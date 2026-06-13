# Testing Plan — Sistem PMB Kampus

| Field | Keterangan |
|---|---|
| Dokumen | Testing Plan (Test Case & Test Scenario) |
| Versi | 1.0 |
| Tanggal | 13 Juni 2026 |
| Referensi | `prd.md`, `devplan.md` |

Dokumen ini berisi **skenario pengujian** dan **test case** untuk memvalidasi seluruh fitur Sistem PMB Kampus, dikelompokkan per modul sesuai PRD.

---

## 1. Strategi & Cakupan Pengujian

| Jenis | Cakupan | Tools (saran) |
|---|---|---|
| Unit Test (FE) | Util, hooks, komponen kecil | Vitest + React Testing Library |
| Unit Test (BE) | Service, validasi, model | PHPUnit / Pest |
| Integration Test (BE) | Endpoint API + DB | Pest + Laravel testing |
| E2E Test | Alur user end-to-end | Playwright / Cypress |
| Manual / UAT | Verifikasi visual & alur bisnis | Checklist manual |
| Integrasi Eksternal | Midtrans (sandbox), Fonnte | Sandbox & nomor uji |

### Prioritas
- **Kritis:** Pendaftaran, validasi NIK unik, pembayaran, notifikasi WA, perubahan status.
- **Tinggi:** Upload dokumen, login, kartu peserta, CRUD admin.
- **Sedang:** Carousel, jadwal, grafik dashboard.

### Format Test Case
| Field | Keterangan |
|---|---|
| ID | Kode unik (mis. `TC-REG-01`) |
| Skenario | Kondisi yang diuji |
| Langkah | Langkah eksekusi |
| Data Uji | Input |
| Hasil Diharapkan | Expected result |
| Prioritas | Kritis / Tinggi / Sedang |

---

## 2. Modul: Landing Page

### TS-LAND — Skenario
Pengunjung melihat informasi kampus, carousel, jadwal, dan program studi.

| ID | Skenario | Langkah | Hasil Diharapkan | Prioritas |
|---|---|---|---|---|
| TC-LAND-01 | Carousel tampil | Buka landing page | Carousel menampilkan slide aktif & auto-slide | Sedang |
| TC-LAND-02 | Carousel kosong | Buka landing saat tidak ada slide aktif | Area carousel tidak error (fallback) | Sedang |
| TC-LAND-03 | Daftar prodi tampil | Buka landing | Program studi aktif tampil | Sedang |
| TC-LAND-04 | Lihat detail jadwal | Klik salah satu jadwal PMB | Halaman detail jadwal tampil | Tinggi |
| TC-LAND-05 | Unduh brosur | Klik tombol unduh pada jadwal | File brosur (PDF) terunduh | Tinggi |
| TC-LAND-06 | Responsif mobile | Buka di viewport mobile | Layout rapi, tanpa overflow | Sedang |

---

## 3. Modul: Pendaftaran

### TS-REG — Skenario
Calon mahasiswa mengisi form pendaftaran; jalur otomatis dari jadwal yang dibuka; pembayaran jika berbayar.

| ID | Skenario | Langkah | Data Uji | Hasil Diharapkan | Prioritas |
|---|---|---|---|---|---|
| TC-REG-01 | Pendaftaran valid (gratis) | Isi form lengkap, submit | Data valid, jalur gratis | Pendaftaran dibuat, status "Terdaftar", WA terkirim (no. pendaftaran & password) | Kritis |
| TC-REG-02 | Pendaftaran valid (berbayar) | Isi form, submit | Data valid, jalur berbayar | Diarahkan ke pembayaran Midtrans | Kritis |
| TC-REG-03 | Jalur otomatis | Buka form pendaftaran | Jadwal aktif tertentu | Field jalur terisi otomatis sesuai jadwal aktif, tidak bisa diubah manual | Kritis |
| TC-REG-04 | NIK duplikat | Daftar dengan NIK yang sudah terdaftar | NIK sudah ada | Pendaftaran ditolak dengan pesan "NIK sudah terdaftar" | Kritis |
| TC-REG-05 | NIK format salah | Isi NIK bukan 16 digit | NIK = "123" | Validasi gagal, pesan error muncul | Tinggi |
| TC-REG-06 | Email tidak valid | Isi email salah format | "abc@" | Validasi gagal | Tinggi |
| TC-REG-07 | No HP tidak valid | Isi no HP non-numerik/terlalu pendek | "08xx" | Validasi gagal | Tinggi |
| TC-REG-08 | Field wajib kosong | Submit form kosong | - | Semua field wajib menampilkan error | Tinggi |
| TC-REG-09 | Tidak ada jadwal dibuka | Buka form saat tidak ada jadwal aktif | - | Form dinonaktifkan / pesan "pendaftaran belum dibuka" | Tinggi |
| TC-REG-10 | Prodi tujuan wajib dipilih | Submit tanpa pilih prodi | - | Validasi gagal | Tinggi |

---

## 4. Modul: Pembayaran (Midtrans Snap)

### TS-PAY — Skenario
Peserta jalur berbayar menyelesaikan pembayaran via Snap; status diperbarui via webhook.

| ID | Skenario | Langkah | Hasil Diharapkan | Prioritas |
|---|---|---|---|---|
| TC-PAY-01 | Buat transaksi Snap | Submit pendaftaran berbayar | Snap token dibuat, popup/redirect Snap tampil | Kritis |
| TC-PAY-02 | Pembayaran sukses (settlement) | Bayar di sandbox sukses | Webhook update status → "Lunas", WA terkirim | Kritis |
| TC-PAY-03 | Pembayaran pending | Pilih metode lalu belum bayar | Status "Menunggu Pembayaran" | Tinggi |
| TC-PAY-04 | Pembayaran expire | Biarkan transaksi kedaluwarsa | Status "expire", peserta bisa membuat ulang (sesuai aturan) | Tinggi |
| TC-PAY-05 | Pembayaran deny/cancel | Transaksi ditolak | Status diperbarui sesuai, tidak menjadi "Lunas" | Tinggi |
| TC-PAY-06 | Webhook signature invalid | Kirim webhook dengan signature salah | Request ditolak (401/403), status tidak berubah | Kritis |
| TC-PAY-07 | Detail pembayaran | Buka detail pembayaran (peserta) | Menampilkan order id, jumlah, status, metode, waktu | Sedang |
| TC-PAY-08 | Idempotensi webhook | Kirim webhook sama 2x | Status tidak terduplikasi/korup | Tinggi |

---

## 5. Modul: Notifikasi WhatsApp (Fonnte)

### TS-WA — Skenario
Sistem mengirim WA otomatis dan manual; pengiriman dicatat.

| ID | Skenario | Langkah | Hasil Diharapkan | Prioritas |
|---|---|---|---|---|
| TC-WA-01 | WA pendaftaran berhasil | Selesaikan pendaftaran gratis/lunas | WA berisi no. pendaftaran & password terkirim | Kritis |
| TC-WA-02 | WA perubahan status | Admin loloskan/tolak pendaftar | WA pemberitahuan status terkirim otomatis | Kritis |
| TC-WA-03 | WA manual admin | Admin kirim notifikasi manual | Pesan terkirim ke peserta terpilih | Tinggi |
| TC-WA-04 | Log notifikasi | Setelah pengiriman | Tercatat di `notifikasi_log` (status kirim) | Tinggi |
| TC-WA-05 | Gagal kirim (retry) | Simulasikan Fonnte error | Job retry / status "gagal" tercatat, tidak crash | Tinggi |
| TC-WA-06 | Nomor WA tidak valid | Kirim ke nomor salah | Ditangani dengan pesan error, dicatat | Sedang |

---

## 6. Modul: Autentikasi & Area Peserta

### TS-AUTH — Skenario

| ID | Skenario | Langkah | Hasil Diharapkan | Prioritas |
|---|---|---|---|---|
| TC-AUTH-01 | Login valid (peserta) | Login no. pendaftaran + password | Berhasil masuk dashboard peserta | Kritis |
| TC-AUTH-02 | Login valid (admin) | Login kredensial admin | Masuk panel admin | Kritis |
| TC-AUTH-03 | Login salah | Password salah | Ditolak dengan pesan error | Tinggi |
| TC-AUTH-04 | Akses tanpa token | Buka halaman terproteksi tanpa login | Redirect ke login (401) | Kritis |
| TC-AUTH-05 | Peserta akses area admin | Login peserta buka URL admin | Ditolak (403) | Kritis |
| TC-AUTH-06 | Logout | Klik logout | Token dihapus, redirect ke login | Tinggi |

### TS-PESERTA — Skenario

| ID | Skenario | Langkah | Hasil Diharapkan | Prioritas |
|---|---|---|---|---|
| TC-PST-01 | Lihat status | Buka dashboard | Status pendaftaran tampil sesuai state | Tinggi |
| TC-PST-02 | Ubah data (diizinkan) | Edit data sebelum dikunci | Data tersimpan | Tinggi |
| TC-PST-03 | Ubah data (terkunci) | Edit setelah diverifikasi | Aksi diblokir/disabled | Tinggi |
| TC-PST-04 | Unduh kartu peserta | Klik unduh kartu | PDF kartu peserta terunduh (data benar) | Tinggi |

---

## 7. Modul: Upload Dokumen (Persyaratan Dinamis)

### TS-DOC — Skenario
Peserta mengunggah dokumen sesuai persyaratan jalur (wajib/opsional, jenis file, ≤10MB).

| ID | Skenario | Data Uji | Hasil Diharapkan | Prioritas |
|---|---|---|---|---|
| TC-DOC-01 | Upload valid (Ijazah PDF) | PDF 2MB | Berhasil, metadata tersimpan | Kritis |
| TC-DOC-02 | Upload valid (Pas Foto PNG) | PNG 1MB | Berhasil | Kritis |
| TC-DOC-03 | File > 10MB | PDF 12MB | Ditolak, pesan "maks 10MB" | Kritis |
| TC-DOC-04 | Tipe file salah | Upload JPG untuk syarat PNG | Ditolak, pesan tipe tidak sesuai | Tinggi |
| TC-DOC-05 | Tipe file salah (PDF) | Upload DOCX untuk syarat PDF | Ditolak | Tinggi |
| TC-DOC-06 | Dokumen opsional kosong | Tidak upload Sertifikat | Pendaftaran tetap dianggap lengkap | Tinggi |
| TC-DOC-07 | Dokumen wajib kosong | Tidak upload Ijazah | Pendaftaran belum bisa "siap verifikasi" | Kritis |
| TC-DOC-08 | Persyaratan dinamis | Jalur dengan syarat custom | Form upload mengikuti konfigurasi jalur | Tinggi |
| TC-DOC-09 | Ganti dokumen | Upload ulang dokumen | File lama tergantikan | Sedang |

---

## 8. Modul: Admin

### TS-ADM-DASH — Dashboard

| ID | Skenario | Hasil Diharapkan | Prioritas |
|---|---|---|---|
| TC-ADM-01 | Summary pendaftar | Kartu ringkasan menampilkan jumlah akurat | Tinggi |
| TC-ADM-02 | Grafik per jalur/prodi | Grafik tampil sesuai data | Sedang |
| TC-ADM-03 | Filter periode | Data menyesuaikan filter | Sedang |

### TS-ADM-DATA — Data Pendaftar & Pembayaran

| ID | Skenario | Hasil Diharapkan | Prioritas |
|---|---|---|---|
| TC-ADM-10 | List pendaftar | Tabel tampil + pagination | Tinggi |
| TC-ADM-11 | Filter & search | Hasil sesuai kriteria | Tinggi |
| TC-ADM-12 | Detail pendaftar | Menampilkan data & dokumen | Tinggi |
| TC-ADM-13 | Ekspor data | File ekspor terunduh | Sedang |
| TC-ADM-14 | Validasi pembayaran manual | Tandai lunas manual | Status diperbarui | Kritis |
| TC-ADM-15 | Detail pembayaran | Menampilkan detail transaksi | Sedang |
| TC-ADM-16 | Unduh kartu peserta (admin) | PDF pendaftar terunduh | Tinggi |

### TS-ADM-STATUS — Ubah Status

| ID | Skenario | Hasil Diharapkan | Prioritas |
|---|---|---|---|
| TC-ADM-20 | Loloskan pendaftar | Status → "Lolos" + WA otomatis terkirim | Kritis |
| TC-ADM-21 | Tolak pendaftar | Status → "Ditolak" + WA otomatis terkirim | Kritis |
| TC-ADM-22 | Ubah status umum | Status diperbarui & tercatat | Tinggi |

### TS-ADM-CRUD — Data Master

| ID | Skenario | Hasil Diharapkan | Prioritas |
|---|---|---|---|
| TC-ADM-30 | CRUD carousel | Tambah/ubah/hapus slide berhasil | Sedang |
| TC-ADM-31 | CRUD program studi | CRUD prodi berhasil | Tinggi |
| TC-ADM-32 | CRUD jalur + persyaratan | CRUD jalur & konfigurasi dokumen berhasil | Tinggi |
| TC-ADM-33 | Persyaratan default | Buat jalur baru | Default (Ijazah PDF wajib, Pas Foto PNG wajib, Sertifikat PDF opsional) otomatis dibuat | Tinggi |
| TC-ADM-34 | CRUD jadwal | CRUD jadwal + set aktif berhasil | Tinggi |
| TC-ADM-35 | Hanya 1 jadwal aktif | Aktifkan jadwal baru | Jalur pendaftaran mengikuti jadwal aktif | Tinggi |
| TC-ADM-36 | CRUD user | CRUD peserta & admin berhasil | Tinggi |

---

## 9. Skenario End-to-End (E2E)

### TS-E2E-01 — Pendaftaran Berbayar (Happy Path)
1. Pengunjung buka landing, lihat jadwal aktif.
2. Isi form pendaftaran (NIK baru) → jalur otomatis (berbayar).
3. Lanjut ke Midtrans, bayar sukses (sandbox).
4. Webhook update status → "Lunas".
5. WA berisi nomor pendaftaran & password diterima.
6. Login peserta, upload dokumen wajib.
7. Admin verifikasi & loloskan → WA "Lolos" terkirim.
8. Peserta unduh kartu peserta.
- **Expected:** seluruh tahap berhasil, status & notifikasi konsisten.

### TS-E2E-02 — Pendaftaran Gratis (Happy Path)
1. Isi form (jalur gratis), submit.
2. Status "Terdaftar", WA no. pendaftaran & password terkirim.
3. Login, upload dokumen, admin loloskan.
- **Expected:** tanpa tahap pembayaran, alur lancar.

### TS-E2E-03 — NIK Duplikat (Negative)
1. Daftar dengan NIK yang sudah ada.
- **Expected:** ditolak, tidak membuat data baru.

### TS-E2E-04 — Pembayaran Gagal (Negative)
1. Daftar berbayar, pembayaran expire/deny.
- **Expected:** status tidak "Lunas", peserta dapat menindaklanjuti sesuai aturan.

---

## 10. Pengujian Non-Fungsional

| ID | Aspek | Skenario | Hasil Diharapkan |
|---|---|---|---|
| TC-NFR-01 | Keamanan | Password tersimpan ter-hash | Tidak ada plaintext di DB |
| TC-NFR-02 | Keamanan | Akses endpoint tanpa otorisasi | Ditolak 401/403 |
| TC-NFR-03 | Keamanan | Verifikasi signature webhook | Request palsu ditolak |
| TC-NFR-04 | Validasi | Input berbahaya (XSS/SQLi) | Disanitasi/diparametrikan |
| TC-NFR-05 | Performa | Tabel data besar | Pagination, respon wajar |
| TC-NFR-06 | Reliabilitas | Kegagalan kirim WA | Retry & log, tidak crash |
| TC-NFR-07 | Responsif | Berbagai ukuran layar | Layout adaptif |

---

## 11. Ringkasan Traceability (Fitur → Test)

| Fitur PRD | Test Case Terkait |
|---|---|
| Landing & carousel | TC-LAND-01..06 |
| Jadwal + unduh brosur | TC-LAND-04, 05 |
| Form pendaftaran + NIK unik | TC-REG-01..10, TC-E2E-03 |
| Jalur otomatis dari jadwal | TC-REG-03, TC-ADM-35 |
| Pembayaran Midtrans | TC-PAY-01..08, TC-E2E-01, 04 |
| Notifikasi WA | TC-WA-01..06, TC-ADM-20, 21 |
| Login & role | TC-AUTH-01..06 |
| Area peserta & kartu | TC-PST-01..04 |
| Upload dokumen dinamis | TC-DOC-01..09 |
| Dashboard & data admin | TC-ADM-01..16 |
| Ubah status | TC-ADM-20..22 |
| CRUD master | TC-ADM-30..36 |
