# PRD — Sistem PMB Kampus (Penerimaan Mahasiswa Baru)

| Field | Keterangan |
|---|---|
| Nama Produk | Sistem PMB Kampus |
| Versi Dokumen | 1.0 |
| Tanggal | 13 Juni 2026 |
| Status | Draft |

---

## 1. Ringkasan (Overview)

Sistem PMB (Penerimaan Mahasiswa Baru) adalah aplikasi web untuk mengelola seluruh proses pendaftaran calon mahasiswa baru, mulai dari menampilkan informasi kampus, proses pendaftaran online, pembayaran, unggah dokumen, hingga pengelolaan & validasi data oleh admin. Sistem juga terintegrasi dengan pembayaran (Midtrans Snap) dan notifikasi WhatsApp (Fonnte).

### 1.1 Tujuan
- Mempermudah calon mahasiswa mendaftar secara online tanpa datang ke kampus.
- Mendukung jalur pendaftaran gratis maupun berbayar.
- Mengotomatiskan notifikasi (nomor pendaftaran, password, status kelulusan) via WhatsApp.
- Memberikan dashboard pemantauan & validasi yang lengkap bagi admin.

### 1.2 Pengguna (User Roles)
| Role | Deskripsi |
|---|---|
| Pengunjung (Guest) | Melihat landing page, jadwal PMB, dan melakukan pendaftaran. |
| Peserta (Pendaftar) | Login untuk melihat status, mengubah data, dan mengunggah dokumen. |
| Admin | Mengelola data master, validasi pembayaran, mengubah status pendaftaran, dan mengirim notifikasi. |

---

## 2. Technology Stack

| Layer | Teknologi |
|---|---|
| Backend (API) | Laravel (PHP) — REST API |
| Frontend | React + shadcn/ui + Tailwind CSS |
| Database | Supabase (PostgreSQL) |
| Penyimpanan File | Supabase Storage (untuk dokumen pendukung) |
| Pembayaran | Midtrans Snap |
| Notifikasi WhatsApp | Fonnte API (`https://docs.fonnte.com/api-send-message/`) |
| Autentikasi | Laravel Sanctum (token-based) |

> Catatan: Frontend React mengonsumsi REST API dari backend Laravel. Database utama menggunakan Supabase (PostgreSQL) yang dikoneksikan ke Laravel.

---

## 3. Fitur & Ruang Lingkup

### 3.1 Halaman Landing & Pendaftaran (Publik)

#### A. Landing Page
- **Carousel informasi kampus** — slideshow gambar/banner berisi informasi tentang kampus (dikelola dari admin).
- **Jadwal PMB** — menampilkan tahapan & timeline pendaftaran.
  - Fitur lihat detail jadwal.
  - Fitur unduh brosur/dokumen jadwal (file PDF).
- **Informasi program studi & jalur pendaftaran**.

#### B. Form Pendaftaran (Alur Utama)
Alur pendaftaran utama:
1. **Isi form data pendaftar:**
   - Nama lengkap
   - NIK
   - No. HP (WhatsApp)
   - Email
   - Jenis kelamin
   - Program studi tujuan
   - **Jalur pendaftaran** — ditentukan **otomatis dari jadwal PMB yang sedang dibuka** (tidak dipilih manual oleh pendaftar).
2. **Pembayaran** — jika jalur yang dibuka memiliki biaya pendaftaran, lanjut ke pembayaran Midtrans Snap. Jika gratis, pendaftaran langsung diproses.

Aturan:
- **Satu NIK hanya boleh memiliki satu pendaftaran (satu jalur).** Sistem memvalidasi keunikan NIK saat pendaftaran; jika NIK sudah terdaftar, pendaftaran ditolak.
- Jalur (beserta biaya & persyaratan dokumen) mengikuti **jadwal PMB yang aktif/dibuka** saat itu.
- Jika berbayar → diarahkan ke **pembayaran Midtrans Snap**.
- Jika gratis → langsung diproses sebagai pendaftaran terdaftar.

#### C. Pembayaran (Midtrans Snap)
- Membuat transaksi Snap untuk biaya pendaftaran jalur berbayar.
- Menangani callback/webhook untuk update status pembayaran (pending, settlement, expire, cancel, deny).
- Menampilkan detail pembayaran ke peserta.

#### D. Notifikasi Pendaftaran (Fonnte)
Setelah pendaftaran berhasil (gratis) atau pembayaran berhasil (berbayar), sistem otomatis mengirim **WhatsApp** ke nomor peserta berisi:
- Nomor pendaftaran.
- Password untuk login ke sistem.

#### E. Area Peserta (Setelah Login)
- Login menggunakan **nomor pendaftaran + password**.
- Melihat **status pendaftaran** (terdaftar, menunggu pembayaran, lunas, sedang diverifikasi, lolos, ditolak).
- **Mengubah data** pendaftaran (selama belum dikunci/diverifikasi).
- **Mengunggah dokumen pendukung** ke Supabase Storage sesuai persyaratan dokumen jalur yang dipilih (lihat bagian 3.3). Maksimal **10 MB** per file, format **PDF atau image** sesuai jenis yang ditentukan tiap persyaratan.
- Melihat status & detail pembayaran.
- **Mengunduh kartu peserta pendaftaran** (PDF) berisi nomor pendaftaran, nama, NIK, prodi tujuan, jalur, dan foto.

### 3.3 Persyaratan Dokumen Dinamis (per Jalur)

Setiap **jalur pendaftaran** memiliki daftar persyaratan dokumen yang dapat dikonfigurasi admin. Tiap persyaratan menentukan:
- **Nama dokumen** (mis. Ijazah Terakhir, Pas Foto, Sertifikat).
- **Wajib / Opsional** — apakah dokumen harus diunggah agar pendaftaran lengkap.
- **Jenis file yang diperbolehkan** — mis. `pdf`, `png`, `jpg/jpeg` (image), atau kombinasi.

Aturan unggah:
- **Ukuran maksimal 10 MB per file.**
- Hanya menerima **PDF atau image** sesuai jenis file yang ditetapkan pada persyaratan.
- Dokumen wajib harus lengkap sebelum pendaftaran dianggap siap diverifikasi.

**Persyaratan default** (saat jalur dibuat, dapat diubah admin):
| Dokumen | Jenis File | Status |
|---|---|---|
| Ijazah Terakhir | PDF | Wajib |
| Pas Foto | PNG | Wajib |
| Sertifikat | PDF | Opsional |

### 3.2 Halaman Admin

- **Dashboard** — grafik & ringkasan (summary) pendaftar dari semua jalur (jumlah pendaftar, status, tren per periode, distribusi prodi).
- **Data Pendaftar** — daftar lengkap pendaftar semua jalur, filter & pencarian, lihat detail, ekspor.
- **Data Pembayaran** — daftar pembayaran semua jalur:
  - **Validasi manual** dan **validasi otomatis** (via webhook Midtrans).
  - Detail pembayaran per transaksi.
- **Manajemen Carousel** — CRUD banner/slide landing page.
- **Manajemen Program Studi** — CRUD program studi.
- **Manajemen Jalur Pendaftaran** — CRUD jalur (termasuk pengaturan gratis/berbayar & nominal) dan **konfigurasi persyaratan dokumen dinamis** per jalur (nama dokumen, wajib/opsional, jenis file). Persyaratan default otomatis dibuat: Ijazah Terakhir (PDF, wajib), Pas Foto (PNG, wajib), Sertifikat (PDF, opsional).
- **Manajemen User** — kelola peserta & admin.
- **Notifikasi WhatsApp** — kirim notifikasi manual ke peserta via Fonnte.
- **Manajemen Status Pendaftaran**:
  - Mengubah status / meloloskan / menolak pendaftaran.
  - Perubahan status **otomatis mengirim notifikasi WhatsApp** ke peserta via Fonnte.
- **Unduh Kartu Peserta** — admin dapat mengunduh kartu peserta (PDF) milik pendaftar mana pun.

---

## 4. Model Data (Ringkasan)

| Entitas | Field Utama |
|---|---|
| `users` | id, role (admin/peserta), nama, email, no_wa, password, nomor_pendaftaran |
| `pendaftar` | id, user_id, nik (**unique**), data_diri (informasi dasar), prodi_pilihan_1, prodi_pilihan_2, jalur_id, status, created_at |
| `program_studi` | id, nama, jenjang, deskripsi, kuota, is_active |
| `jalur_pendaftaran` | id, nama, deskripsi, is_berbayar, biaya, periode_mulai, periode_selesai, is_active |
| `persyaratan_dokumen` | id, jalur_id, nama_dokumen, jenis_file (pdf/png/jpg/...), is_wajib, urutan |
| `pembayaran` | id, pendaftar_id, order_id, jumlah, status, metode, snap_token, validasi_manual, paid_at |
| `dokumen` | id, pendaftar_id, persyaratan_id, nama, file_url, ukuran, mime_type, status_verifikasi |
| `carousel` | id, judul, gambar_url, link, urutan, is_active |
| `jadwal_pmb` | id, judul, jalur_id, tanggal_mulai, tanggal_selesai, deskripsi, brosur_url, is_active |
| `notifikasi_log` | id, pendaftar_id, pesan, status_kirim, created_at |

---

## 5. Status Pendaftaran (State)

```
Terdaftar → Menunggu Pembayaran → Lunas → Verifikasi Dokumen → Lolos / Ditolak
```
- Jalur gratis melewati state "Menunggu Pembayaran" & "Lunas".
- Setiap perubahan ke status final (Lolos/Ditolak) memicu notifikasi WA otomatis.

---

## 6. Roadmap (3 Fase)

### Fase 1 — Halaman Landing & Pendaftaran (Frontend)
**Tujuan:** Membangun seluruh tampilan publik & alur peserta menggunakan React + shadcn/ui (data mock/dummy lebih dulu bila API belum siap).

- [ ] Setup project React + shadcn/ui + Tailwind + routing.
- [ ] Landing page: carousel, info kampus.
- [ ] Komponen jadwal PMB: list, detail, tombol unduh brosur.
- [ ] Form pendaftaran multi-step (informasi dasar, prodi, jalur).
- [ ] Halaman pemilihan jalur (gratis/berbayar) + ringkasan biaya.
- [ ] Halaman pembayaran (integrasi UI Midtrans Snap).
- [ ] Halaman login peserta.
- [ ] Dashboard peserta: status pendaftaran, ubah data, unggah dokumen, detail pembayaran, unduh kartu peserta (PDF).
- [ ] Responsive & UI/UX modern.

### Fase 2 — Halaman Admin (Frontend)
**Tujuan:** Membangun seluruh antarmuka admin.

- [ ] Layout admin (sidebar, auth guard).
- [ ] Dashboard: grafik & summary pendaftar.
- [ ] Tabel data pendaftar (filter, search, detail, ekspor).
- [ ] Tabel data pembayaran (validasi manual & detail).
- [ ] CRUD carousel.
- [ ] CRUD program studi.
- [ ] CRUD jalur pendaftaran.
- [ ] CRUD user (peserta & admin).
- [ ] Form kirim notifikasi WA.
- [ ] Aksi ubah status / loloskan / tolak pendaftaran.
- [ ] Unduh kartu peserta (PDF) per pendaftar.

### Fase 3 — Integrasi Backend (Laravel + Supabase)
**Tujuan:** Membangun API & mengintegrasikan seluruh frontend dengan backend, pembayaran, dan notifikasi.

- [ ] Setup Laravel + koneksi Supabase (PostgreSQL).
- [ ] Migrasi & model database (sesuai bagian 4).
- [ ] Autentikasi (Sanctum) untuk peserta & admin.
- [ ] API pendaftaran, prodi, jalur, jadwal, carousel.
- [ ] Integrasi Midtrans Snap (create transaksi + webhook callback).
- [ ] Integrasi Fonnte (kirim WA: nomor pendaftaran & password, perubahan status).
- [ ] Upload dokumen ke Supabase Storage.
- [ ] API dashboard & summary admin.
- [ ] Validasi pembayaran manual & otomatis.
- [ ] Sambungkan seluruh frontend Fase 1 & 2 ke API.

---

## 7. Integrasi Eksternal

### 7.1 Midtrans Snap
- Membuat Snap token di backend untuk biaya jalur berbayar.
- Webhook menerima notifikasi status & memperbarui tabel `pembayaran`.
- Status sukses memicu notifikasi WA + update status pendaftaran.

### 7.2 Fonnte (WhatsApp)
- Endpoint: `https://docs.fonnte.com/api-send-message/`.
- Pemicu otomatis:
  1. Pendaftaran berhasil → kirim nomor pendaftaran & password.
  2. Perubahan status (lolos/ditolak) → kirim pemberitahuan.
- Pemicu manual: admin mengirim notifikasi dari panel.
- Semua pengiriman dicatat di `notifikasi_log`.

---

## 8. Persyaratan Non-Fungsional
- **Keamanan:** password di-hash, otorisasi berbasis role, validasi input, verifikasi webhook (signature Midtrans).
- **Responsif:** mendukung desktop & mobile.
- **Performa:** pagination pada tabel data besar.
- **Reliabilitas:** retry/log untuk kegagalan kirim WA & webhook pembayaran.

---

## 9. Asumsi & Keputusan
- Versi Laravel mengikuti versi LTS/stabil terbaru yang tersedia.
- **Satu NIK = satu pendaftaran (satu jalur).** NIK divalidasi unik saat mendaftar.
- **Unggah dokumen:** maksimal 10 MB per file, format PDF atau image sesuai jenis pada tiap persyaratan.
- **Persyaratan dokumen bersifat dinamis per jalur** (wajib/opsional, jenis file). Default: Ijazah Terakhir (PDF, wajib), Pas Foto (PNG, wajib), Sertifikat (PDF, opsional).
- **Jalur pendaftaran ditentukan otomatis dari jadwal PMB yang aktif/dibuka**, bukan dipilih manual oleh pendaftar.
- **Kartu peserta pendaftaran (PDF) dapat diunduh oleh peserta maupun admin.**

### Pertanyaan Terbuka
- _(Belum ada — semua poin utama sudah ditentukan.)_
