# Kebijakan Keamanan

## Versi yang Didukung

Kami berkomitmen menjaga keamanan proyek-proyek di organisasi Webekspres. Versi yang saat ini menerima update keamanan:

| Proyek | Versi Didukung |
|--------|----------------|
| Plugin WordPress aktif | Versi terbaru |
| Aplikasi web internal | Versi production aktif |

> Untuk proyek spesifik, periksa file `SECURITY.md` di repositori masing-masing jika tersedia.

---

## Melaporkan Kerentanan

Kami sangat menghargai laporan kerentanan keamanan dari komunitas. Jika Anda menemukan celah keamanan, **jangan** buka issue publik.

### Cara Melaporkan

Kirim laporan ke: **data.webekspres@gmail.com**

Subjek email: `[SECURITY] <nama proyek> - deskripsi singkat`

### Informasi yang Disertakan

Agar kami dapat menindaklanjuti dengan cepat, sertakan:

1. **Deskripsi kerentanan** — jelaskan apa yang Anda temukan
2. **Langkah reproduksi** — cara memicu kerentanan
3. **Dampak potensial** — apa yang bisa dilakukan penyerang
4. **Environment** — versi software, browser, OS yang digunakan
5. **Proof of Concept** — jika ada (jangan sertakan data klien nyata)
6. **Kontak Anda** — untuk follow-up (opsional, bisa anonim)

### Apa yang Diharapkan

| Timeline | Tindakan |
|----------|----------|
| **24 jam** | Konfirmasi penerimaan laporan |
| **72 jam** | Assessment awal dan klasifikasi severity |
| **7–30 hari** | Perbaikan tergantung tingkat keparahan |
| **Setelah fix** | Koordinasi publikasi advisory (jika diperlukan) |

---

## Praktik Keamanan Internal

Tim engineering Webekspres menerapkan:

- Validasi dan sanitasi input di semua layer
- Principle of least privilege untuk akses database dan server
- Tidak menyimpan credentials di repositori Git
- Regular dependency updates untuk plugin dan library
- HTTPS wajib untuk semua deployment production

---

## Hall of Fame

Kami mengucapkan terima kasih kepada peneliti keamanan yang telah melaporkan kerentanan secara bertanggung jawab. Daftar akan dipublikasikan dengan izin pelapor.

---

## Disclaimer

Informasi di halaman ini berlaku untuk proyek yang dikelola organisasi [Webekspres](https://github.com/Webekspres). Proyek pihak ketiga atau fork mungkin memiliki kebijakan keamanan tersendiri.
