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

Organisasi memakai runner self-hosted di 3 PC kantor (grup `Default`, hanya repo **privat**).

| Runner | Mesin | Label | Peran |
|---|---|---|---|
| `forge`, `forge-2`, `forge-3` | pc-padli (16 GB) | `self-hosted, heavy, docker` | job berat: test, build, deploy |
| `atlas`, `atlas-2`, `atlas-3` | kris-lan (16 GB, Fedora + SELinux) | `self-hosted, heavy, docker` | job berat |
| `spark` | laptop 4 GB | `self-hosted, light, docker` | hanya job ringan (lint, cek terjemahan) |

Repo **publik** dilarang memakai runner self-hosted (keamanan: PR dari fork bisa menjalankan kode di PC kantor).
Repo publik memakai `ubuntu-latest` (gratis, tanpa batas).

### Label = kemampuan runner, bukan versi tools

Label hanya menyatakan apa yang **bisa** dilakukan runner. Versi PHP/Node/Python **tidak** ditentukan oleh runner,
melainkan oleh image kontainer di workflow:

| Kebutuhan job | `runs-on` |
|---|---|
| Test/lint biasa (boleh di spark) | `[self-hosted, docker]` |
| Test berat, build, E2E | `[self-hosted, heavy, docker]` |
| Build image Docker / deploy | `[self-hosted, heavy, docker]` (memakai Docker milik runner) |

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
    runs-on: [self-hosted, heavy, docker]
    container:
      image: ghcr.io/webekspres/ci-php:8.4
      options: --security-opt label=disable   # wajib: atlas memakai SELinux
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
      - name: Kembalikan kepemilikan workspace ke user runner
        if: always()
        run: chown -R "$(stat -c %u:%g "$GITHUB_WORKSPACE/..")" "$GITHUB_WORKSPACE"
```

Aturan:

- **Jangan** memasang tools ke PC runner (setup-php dengan sudo, apt install, symlink). Butuh tools baru: tambahkan ke ci-images.
- Versi Bun/Node/uv yang dipatok repo boleh tetap lewat `setup-bun` / `setup-node` / `setup-uv` **di dalam** kontainer.
- Service (MySQL/Postgres/Redis) diakses lewat **nama service**, tanpa `ports:`.
- Langkah terakhir `chown` wajib: kontainer berjalan sebagai root, tanpa itu job berikutnya di runner yang sama gagal checkout.
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
