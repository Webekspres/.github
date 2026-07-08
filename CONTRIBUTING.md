# Panduan Kontribusi

Terima kasih atas minat Anda untuk berkontribusi pada proyek **Webekspres**. Dokumen ini menjelaskan standar dan alur kerja yang kami gunakan di seluruh organisasi.

---

## Prasyarat

Sebelum berkontribusi, pastikan Anda:

- [ ] Memiliki akses ke repositori terkait di organisasi [Webekspres](https://github.com/Webekspres)
- [ ] Membaca [Kode Etik](CODE_OF_CONDUCT.md) kami
- [ ] Memahami stack teknologi proyek yang Anda kerjakan
- [ ] Mengetahui [panduan deployment](DEPLOYMENT_GUIDE.md) jika mendeploy ke production

---

## Alur Kerja Git

Kami menggunakan **Git Flow** yang disederhanakan:

```
main          → kode production-ready
develop       → integrasi fitur (jika ada)
feature/*     → pengembangan fitur baru
fix/*         → perbaikan bug
hotfix/*      → perbaikan darurat di production
```

### Langkah Kontribusi

1. **Fork / Clone** repositori target
2. **Buat branch** dari `main` (atau `develop` jika ada):
   ```bash
   git checkout -b feature/nama-fitur
   ```
3. **Commit** dengan pesan yang jelas (lihat [Conventional Commits](#conventional-commits))
4. **Push** branch Anda
5. **Buat Pull Request** menggunakan template yang tersedia
6. **Tunggu review** dari maintainer

---

## Conventional Commits

Gunakan format commit message berikut:

```
<type>(<scope>): <deskripsi singkat>

[body opsional]
```

| Type | Keterangan |
|------|------------|
| `feat` | Fitur baru |
| `fix` | Perbaikan bug |
| `docs` | Perubahan dokumentasi |
| `style` | Formatting, tanpa perubahan logika |
| `refactor` | Refactoring tanpa fitur/bug fix |
| `test` | Menambah atau memperbaiki test |
| `chore` | Maintenance, dependency update |

**Contoh:**
```
feat(woocommerce): tambah notifikasi WhatsApp saat order selesai
fix(api): perbaiki validasi token yang expired
docs(readme): update instruksi instalasi plugin
```

---

## Standar Kode

### PHP / Laravel / WordPress

- Ikuti [PSR-12](https://www.php-fig.org/psr/psr-12/) untuk style guide PHP
- Gunakan prefix unik untuk fungsi/hook WordPress (hindari konflik namespace)
- Validasi dan sanitasi semua input pengguna
- Escape semua output ke browser

### JavaScript / TypeScript / React

- Gunakan ESLint dan Prettier sesuai konfigurasi proyek
- Prefer functional components dan hooks di React
- Hindari `any` di TypeScript kecuali benar-benar diperlukan

### Umum

- Tulis kode yang **readable** — prioritas di atas clever code
- Satu fungsi, satu tanggung jawab
- Komentar hanya untuk logika yang tidak obvious
- Jangan commit file sensitif (`.env`, credentials, API keys)

---

## Pull Request

Sebelum submit PR, pastikan:

- [ ] Branch sudah up-to-date dengan base branch
- [ ] Kode sudah ditest secara manual (atau automated test jika ada)
- [ ] Tidak ada conflict dengan branch target
- [ ] Deskripsi PR menjelaskan **apa** dan **mengapa** perubahan dilakukan
- [ ] Screenshot disertakan untuk perubahan UI

### Review Process

- Minimal **1 approval** dari maintainer sebelum merge
- Gunakan **Squash Merge** untuk fitur kecil
- Gunakan **Merge Commit** untuk release besar

---

## Pelaporan Bug

Gunakan [template bug report](ISSUE_TEMPLATE/bug_report.yml) saat membuka issue. Sertakan:

- Langkah reproduksi
- Perilaku yang diharapkan vs aktual
- Environment (OS, browser, versi PHP, dll.)
- Screenshot atau log error jika ada

---

## Permintaan Fitur

Gunakan [template feature request](ISSUE_TEMPLATE/feature_request.yml). Jelaskan:

- Masalah yang ingin diselesaikan
- Solusi yang diusulkan
- Alternatif yang sudah dipertimbangkan

---

## Deployment

Sebelum mendeploy ke production, ikuti [Panduan Deployment](DEPLOYMENT_GUIDE.md) organisasi:

1. Daftarkan subdomain di Plesk terlebih dahulu
2. Pilih metode deployment sesuai stack project (Vercel, cPanel, VPS Docker, atau Plesk)
3. Jangan commit file `.env` atau kredensial ke Git — lihat [Kebijakan Keamanan](SECURITY.md)

---

## Pertanyaan?

- **Email:** data.webekspres@gmail.com
- **Website:** [webekspres.co.id](https://webekspres.co.id)
- Lihat juga [SUPPORT.md](SUPPORT.md) untuk panduan bantuan

---

<div align="center">

**PT Webekspres Teknologi Indonesia**

</div>
