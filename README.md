<div align="center">

<img width="4010" height="1127" alt="Frame 427320582" src="https://github.com/user-attachments/assets/8a44720a-a8b4-4a13-9461-52ba5112c838" />


# Repositori `.github` Organisasi Webekspres

**Profil organisasi publik dan file default community health untuk [Webekspres](https://github.com/Webekspres)**

[![Website](https://img.shields.io/badge/Website-webekspres.co.id-0f3460?style=for-the-badge&logo=googlechrome&logoColor=white)](https://webekspres.co.id)
[![Email](https://img.shields.io/badge/Email-data.webekspres%40gmail.com-1a1a2e?style=for-the-badge&logo=gmail&logoColor=white)](mailto:data.webekspres@gmail.com)

</div>

---

## Tentang Repositori Ini

Repositori **publik** ini adalah sumber konfigurasi tingkat organisasi untuk **PT Webekspres Teknologi Indonesia**. GitHub menggunakan repo bernama `.github` untuk menampilkan **profil organisasi publik** serta menyediakan **file default** yang otomatis berlaku di seluruh repositori organisasi yang belum memiliki file serupa.

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

### Workflow template

| Template | Untuk |
|---|---|
| Webekspres CI Laravel / Node / Python / Flutter (repo privat) | tes di kontainer image [`ci-images`](https://github.com/Webekspres/ci-images) pada runner organisasi |
| Webekspres Deploy Docker ke VPS (repo privat) | build & push image ke GHCR lalu deploy, hanya dari `staging` dan `main` |
| Webekspres Alur Branch (repo privat / repo publik) | menjaga alur `fitur → dev → staging → main` |

Template bertanda **repo privat** memakai runner organisasi dan hanya boleh dipakai di repo privat.
Repo publik selalu memakai runner GitHub (`ubuntu-latest`).

### Action `tunggu-db`

```yaml
- uses: Webekspres/.github/actions/tunggu-db@main
  with:
    type: mysql      # mysql | mariadb | postgres | redis
    password: root
```

Detail input ada di [`actions/tunggu-db/README.md`](actions/tunggu-db/README.md).

Dokumentasi internal (infrastruktur, CI/runner, deploy, handover) **tidak** disimpan di sini karena repo ini publik.
Tempatnya di repo privat [`.github-private`](https://github.com/Webekspres/.github-private), yang juga berisi profil khusus anggota.

---

## Profil Organisasi

| Tampilan | Repositori | Visibility |
|---|---|---|
| **Publik** (semua pengunjung) | `.github` (repo ini) | Public |
| **Member** (khusus anggota org) | `.github-private` | Private |

Kedua repositori memiliki `profile/README.md` masing-masing. GitHub menampilkan versi publik dari repo ini kepada semua orang, dan versi member dari `.github-private` kepada anggota organisasi.

---

<div align="center">

**PT Webekspres Teknologi Indonesia** · Indonesia

![Footer](https://capsule-render.vercel.app/api?type=waving&color=0:0f3460,100:1a1a2e&height=80&section=footer)

</div>
