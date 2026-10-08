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

| Item | Nilai |
|---|---|
| Label | `[self-hosted, devserver]` |
| Cakupan | grup `Default`, seluruh repo **privat** organisasi |
| Repo publik | **dilarang**. Pakai `ubuntu-latest` (`fonte-otp`, `buddypress-activity-approval`, `pixel-komunika`, `glubee.id`, `.github`) |
| Jumlah | 1 runner, jadi job berjalan bergantian. Hindari job panjang di `dev` |

Aturan pemakaian:

- Job deploy tidak boleh dibatalkan di tengah jalan (`cancel-in-progress: false` di `staging` dan `main`).
- Jangan memicu dua workflow berat sekaligus untuk perubahan yang sama (mis. `push` dan `pull_request` ke branch yang sama untuk tes yang sama).
- Jangan menyimpan secret di repo. Gunakan GitHub Secrets (per environment `staging` / `production`) atau Doppler.

## Menambah atau mengubah workflow

1. Salin template trigger di atas.
2. Tentukan job per branch dengan `if: github.ref == 'refs/heads/...'` (mis. build dan deploy tidak jalan di `dev`).
3. Pastikan `runs-on: [self-hosted, devserver]` hanya dipakai di repo privat.
4. Uji lewat `workflow_dispatch` sebelum merge ke `staging`.
