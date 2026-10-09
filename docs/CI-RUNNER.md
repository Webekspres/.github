# CI/CD & Self-hosted Runner

Standar CI untuk seluruh repositori organisasi Webekspres. Dokumen ini melengkapi `docs/BRANCHING.md` di tiap repo.

## Prinsip

1. CI hanya berjalan di tiga branch: `dev`, `staging`, `main`. Branch lain tidak memakai runner.
2. Makin dekat ke production, makin lengkap tes dan makin berhati-hati deploy-nya.
3. Perubahan dokumentasi saja (`**.md`, `docs/**`) tidak memicu CI.
4. Runner dibagi untuk seluruh organisasi, jadi hemat: batalkan run lama di `dev`, jangan batalkan deploy.

## Matriks tes per branch

| Branch | Trigger | Isi pipeline | Concurrency |
|---|---|---|---|
| `dev` | `push` dan `pull_request` ke `dev` | lint, typecheck, unit test (tanpa build image, tanpa deploy) | `cancel-in-progress: true` |
| `staging` | `push` | seluruh tes (unit, integration/e2e), build image `:staging`, deploy staging | `cancel-in-progress: false` |
| `main` | `push` | build image `:latest`, deploy production (tes sudah lolos di staging; test cepat tetap dijalankan) | `cancel-in-progress: false` |
| lainnya | tidak ada | tidak ada CI. Gunakan `workflow_dispatch` bila perlu menjalankan manual | - |

Template trigger:

```yaml
on:
  push:
    branches: [dev, staging, main]
    paths-ignore: ['**.md', 'docs/**']
  pull_request:
    branches: [dev, staging, main]
    paths-ignore: ['**.md', 'docs/**']
  workflow_dispatch:

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: ${{ github.ref == 'refs/heads/dev' || github.event_name == 'pull_request' }}
```

Hindari `branches: ['**']` dan `branches-ignore`, karena keduanya memakai runner di semua branch fitur.

## Runner

Dua jenis runner, semua di jaringan kantor (grup `Default`, hanya repo **privat**):

### 1. ARC: runner sekali pakai di Kubernetes (untuk job berat)

Cluster k3s di **kris-lan** (server) + **pc-padli** (agent). GitHub mengirim job, ARC membuat
**pod baru khusus job itu** di PC yang sedang longgar, lalu **menghapusnya** setelah selesai.
Tidak ada sisa file atau tools yang menumpuk di PC.

| `runs-on` | Untuk | Kapasitas |
|---|---|---|
| `arc-ci` | test, lint berat, build aplikasi, E2E, APK (wajib `container:`) | 0–6 job paralel |
| `arc-docker` | job yang menjalankan `docker build` (Docker terisolasi per job) | 0–3 job paralel |

Yang diatur otomatis oleh template pod ARC (tidak perlu ditulis di workflow):
- nama service `mysql`, `postgres`, `redis`, `mariadb` mengarah ke service job (sama seperti di Docker)
- batas memori 8 GB per job, supaya satu job tidak membuat PC macet
- cache dependensi per PC di `/ci-cache` (composer, npm, bun, gradle, uv, go, playwright)

### 2. Runner biasa (untuk job ringan dan deploy)

| Runner | Mesin | Label |
|---|---|---|
| `spark` | laptop 4 GB | `self-hosted, light, docker` |
| `forge`, `forge-2`, `forge-3` | pc-padli | `self-hosted, heavy, docker` |
| `atlas`, `atlas-2`, `atlas-3` | kris-lan | `self-hosted, heavy, docker` |

Runner forge/atlas akan dipensiunkan setelah semua job berat dan deploy pindah ke ARC.
Spark tetap untuk job ringan (lint, cek terjemahan) dengan `runs-on: [self-hosted, docker]`.

Repo **publik** dilarang memakai runner kantor (keamanan: PR dari fork bisa menjalankan kode di PC kantor).
Repo publik memakai `ubuntu-latest` (gratis, tanpa batas).

### Label = kemampuan runner, bukan versi tools

Label hanya menyatakan apa yang **bisa** dilakukan runner. Versi PHP/Node/Python **tidak** ditentukan oleh runner,
melainkan oleh image kontainer di workflow:

| Kebutuhan job | `runs-on` |
|---|---|
| Lint / cek ringan (boleh di spark) | `[self-hosted, docker]` |
| Test berat, build, E2E, APK | `arc-ci` |
| Build image Docker | `arc-docker` |
| Deploy ke VPS (sementara) | `[self-hosted, heavy, docker]` |

Label lama `php`, `dbports`, dan `devserver` **jangan dipakai** di workflow baru; akan dicabut setelah semua branch `main` diperbarui.

### Job berjalan di kontainer (wajib untuk test/lint)

Tools diambil dari image [Webekspres/ci-images](https://github.com/Webekspres/ci-images), bukan dari PC:

| Image | Untuk |
|---|---|
| `ghcr.io/webekspres/ci-php:8.3`, `:8.4` | Laravel/Bagisto: PHP + ekstensi, composer, Node 22, Bun |
| `ghcr.io/webekspres/ci-node:22` | Next.js, Vite, Prisma, Bun |
| `ghcr.io/webekspres/ci-python:3.12` | Django/FastAPI dengan uv |
| `ghcr.io/webekspres/ci-flutter:3.44.6` | Flutter + Android SDK + JDK 17 |

Contoh lengkap:

```yaml
jobs:
  test:
    runs-on: arc-ci
    container:
      image: ghcr.io/webekspres/ci-php:8.4
    services:
      mysql:
        image: mysql:8.4
        env: { MYSQL_ROOT_PASSWORD: root, MYSQL_DATABASE: app }
        # tanpa "ports:" -> beberapa job bisa paralel di satu PC
    env:
      DB_HOST: mysql      # nama service, BUKAN 127.0.0.1/localhost
      DB_PORT: 3306
    steps:
      - uses: actions/checkout@v4
      - run: composer install && php artisan test
```

Untuk job di runner biasa (`[self-hosted, docker]`) tambahkan di `container:`
`options: --security-opt label=disable` (atlas memakai SELinux) dan langkah terakhir:

```yaml
      - name: Kembalikan kepemilikan workspace ke user runner
        if: always()
        run: chown -R "$(stat -c %u:%g "$GITHUB_WORKSPACE/..")" "$GITHUB_WORKSPACE"
```

Aturan:

- **Jangan** memasang tools ke PC runner (setup-php dengan sudo, apt install, symlink). Butuh tools baru: tambahkan ke ci-images.
- Versi Bun/Node/uv yang dipatok repo boleh tetap lewat `setup-bun` / `setup-node` / `setup-uv` **di dalam** kontainer.
- Service (MySQL/Postgres/Redis) diakses lewat **nama service**, tanpa `ports:`.
- Runner biasa: langkah `chown` wajib (kontainer berjalan sebagai root). Di ARC tidak perlu.
- Stack baru: buat `images/<nama>/Dockerfile` di ci-images dan tambahkan ke matrix build-nya.

### Penggunaan bersama

- Job deploy tidak boleh dibatalkan di tengah jalan (`cancel-in-progress: false` di `staging` dan `main`).
- Jangan memicu dua workflow berat sekaligus untuk perubahan yang sama.
- Jangan menyimpan secret di repo. Gunakan GitHub Secrets (per environment `staging` / `production`).
- Jangan memakai cache GitHub (`actions/cache`, `cache: npm`, `cache-to: type=gha`) di runner self-hosted:
  unggahan dari jaringan kantor sangat lambat. Cache layer Docker lokal sudah cukup.

Pantau runner dan antrean dari terminal dengan [pitwall](https://github.com/Webekspres/pitwall).

## Menambah atau mengubah workflow

1. Salin template trigger di atas.
2. Tentukan job per branch dengan `if: github.ref == 'refs/heads/...'` (mis. build dan deploy tidak jalan di `dev`).
3. Test/lint: `container:` dari ci-images + `runs-on` sesuai tabel di atas.
4. Uji lewat `workflow_dispatch` atau PR ke `dev` sebelum merge ke `staging`.

## Mengelola cluster ARC

Konfigurasi ada di `~/arc/` pada kris-lan (salinan: `webekspres-ci/arc/` di PC admin). Akses: `ssh kris-lan`, lalu `kubectl`.

| Kebutuhan | Perintah |
|---|---|
| Lihat node | `kubectl get nodes` |
| Lihat runner yang sedang bekerja | `kubectl get pods -n arc-runners` |
| Ubah jumlah maksimum runner | ubah `maxRunners` di `values-arc-ci.yaml`, lalu `helm upgrade arc-ci -n arc-runners -f values-arc-ci.yaml oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set` |
| Ubah template job (memori, cache, nama service) | ubah `job-pod-template.yaml`, lalu `kubectl -n arc-runners create configmap arc-job-pod-template --from-file=job.yaml=job-pod-template.yaml --dry-run=client -o yaml \| kubectl apply -f -` |
| Uji kesehatan ARC | repo pitwall → Actions → **ARC smoke** → Run workflow |
| Tambah PC ke cluster | jalankan `02-k3s-agent-pc-padli.sh` (ganti IP) di PC baru |

Kunci GitHub App `webekspres-arc` disimpan sebagai secret `arc-github-app` di namespace `arc-runners`, bukan di repo mana pun.
