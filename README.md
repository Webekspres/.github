# Webekspres `.github`

Profil organisasi publik, file default komunitas, serta workflow dan action CI bersama untuk
[Webekspres](https://github.com/Webekspres). Repo ini **wajib publik**.

## Tentang Repositori Ini

Repo ini punya tiga peran yang berbeda dan saling lepas:

1. **Profil organisasi publik.** GitHub hanya membaca [`profile/README.md`](profile/README.md) untuk halaman
   [github.com/Webekspres](https://github.com/Webekspres). `README.md` yang sedang Anda baca **tidak** tampil di sana.
2. **File default komunitas.** CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, SUPPORT, dan template issue/PR berlaku
   otomatis untuk setiap repo organisasi yang belum punya file sendiri.
3. **CI bersama.** Reusable workflow `build-image` (build image Docker ke GHCR) dan `deploy-ssh`, action
   `tunggu-db`, serta template workflow. Repo lain memanggilnya lewat tag `@v1`, jadi mengubah README atau profil
   tidak memengaruhi CI.

| File / Folder | Fungsi |
|---|---|
| [`profile/README.md`](profile/README.md) | Profil organisasi yang tampil di [github.com/Webekspres](https://github.com/Webekspres) |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Panduan kontribusi dan standar engineering |
| [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md) | Kode etik kolaborasi |
| [`SECURITY.md`](SECURITY.md) | Kebijakan pelaporan kerentanan keamanan |
| [`SUPPORT.md`](SUPPORT.md) | Cara mendapatkan bantuan |
| [`ISSUE_TEMPLATE/`](ISSUE_TEMPLATE/) | Template issue default untuk repositori org |
| [`PULL_REQUEST_TEMPLATE.md`](PULL_REQUEST_TEMPLATE.md) | Template pull request default |
| [`workflow-templates/`](workflow-templates/) | Template GitHub Actions organisasi (tombol **Actions → New workflow** di setiap repo) |
| [`actions/tunggu-db/`](actions/tunggu-db/) | Action bersama: tunggu MySQL/MariaDB/Postgres/Redis siap sebelum tes |
| [`.github/workflows/`](.github/workflows/) | Reusable workflow `build-image` dan `deploy-ssh` yang dipanggil `ci-cd.yml` setiap repo |

### Workflow template

| Template | Untuk |
|---|---|
| **Webekspres CI/CD standar (repo privat)** | alur utama setiap proyek: test → build image GHCR → deploy staging / production |
| Webekspres CI Laravel / Node / Python / Flutter (repo privat) | contoh job test lengkap per stack (service database, `tunggu-db`) untuk diisi ke `ci-cd.yml` |
| Webekspres Alur Branch (repo privat / repo publik) | menjaga alur `fitur → dev → staging → main` |

Template bertanda **repo privat** memakai runner organisasi dan hanya boleh dipakai di repo privat.
Repo publik selalu memakai runner GitHub (`ubuntu-latest`).

### Alur CI/CD standar

Setiap repo privat punya satu `.github/workflows/ci-cd.yml` dengan bentuk yang sama:

| Kejadian | Job |
|---|---|
| PR ke `dev` / `staging` / `main`, push `dev` | test |
| push `staging` | test → build (`:staging`, `:sha-<commit>`) → deploy-staging |
| push `main` | test → build (`:latest`, `:sha-<commit>`) → deploy-production |

Job test khas proyek dan ditulis di repo. Build dan deploy memanggil reusable workflow di repo ini:

```yaml
  build:
    needs: test
    if: github.event_name != 'pull_request' && (github.ref_name == 'staging' || github.ref_name == 'main')
    uses: Webekspres/.github/.github/workflows/build-image.yml@v1
    with:
      environment: ${{ github.ref_name == 'main' && 'production' || 'staging' }}

  deploy-staging:
    needs: build
    if: github.ref_name == 'staging'
    uses: Webekspres/.github/.github/workflows/deploy-ssh.yml@v1
    with:
      environment: staging
      image-tag: ${{ needs.build.outputs.tag }}
      secret-prefix: STAGING_
      script: |
        cd /opt/app-staging && docker compose pull app && docker compose up -d app
    secrets: inherit
```

Input lengkap ada di komentar [`build-image.yml`](.github/workflows/build-image.yml) dan
[`deploy-ssh.yml`](.github/workflows/deploy-ssh.yml).

**Versi.** Repo memanggil `@v1`. Perbaikan yang tidak mengubah input memindahkan tag `v1` sehingga semua repo
langsung ikut; perubahan yang tidak kompatibel dirilis sebagai `v2` dan tiap repo pindah lewat PR.
Versi runtime proyek (PHP, Node, Python, Flutter, database) tetap ditentukan di job test repo masing-masing
dan harus sama dengan versi repo, dev, dan produksi.

### Action `tunggu-db`

```yaml
- uses: Webekspres/.github/actions/tunggu-db@v1
  with:
    type: mysql      # mysql | mariadb | postgres | redis
    password: root
```

Detail input ada di [`actions/tunggu-db/README.md`](actions/tunggu-db/README.md).

Dokumentasi internal (infrastruktur, CI/runner, deploy, handover) **tidak** disimpan di sini karena repo ini publik.
Tempatnya di repo privat [`.github-private`](https://github.com/Webekspres/.github-private), yang juga berisi profil khusus anggota.

---

## Profil Organisasi

| Tampilan | File | Dilihat oleh |
|---|---|---|
| Publik | [`.github/profile/README.md`](profile/README.md) (repo ini) | semua pengunjung |
| Anggota | `.github-private/profile/README.md` | anggota org (pilih *View as: Member*) |

Profil publik memperkenalkan perusahaan. Profil anggota adalah titik awal kerja anggota: langkah awal, alur rilis, CI/CD,
dokumentasi internal, peta repositori, dan FAQ. Isinya memang berbeda; jangan menyalin satu ke yang lain.

Aturan saat mengubah profil:

- Pakai URL lengkap (`https://github.com/Webekspres/...`) untuk tautan dan gambar, supaya hasilnya sama di
  halaman organisasi dan saat file dibuka langsung.
- Hindari widget pihak ketiga (penghitung kunjungan, banner gelombang, badge berwarna gelap): sering gagal
  dimuat dan tidak terbaca di tema gelap GitHub. Teks dan tabel Markdown biasa selalu tampil.
- Gambar yang diunggah lewat repo privat hanya terlihat oleh anggota. Gambar profil publik diunggah dari repo ini.
