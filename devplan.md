# Development Plan — Sistem Penerimaan Mahasiswa Baru

> Diturunkan dari `prd.md`. Fokus pada 4 alur inti:
> **Pendaftaran → Pelengkapan Pendaftaran → Penilaian → Pengumuman.**
> Stack: React 18 + Tailwind (frontend) · Laravel 12 + Sanctum (backend) · SQLite (dev) / PostgreSQL (prod).

---

## 0. Ringkasan Alur End-to-End

```
[Calon Mahasiswa]                         [Admin / Panitia]
       │                                          │
       ▼                                          │
 1. PENDAFTARAN  ──────────────────────────────►  │
   (isi biodata, dapat nomor pendaftaran)         │
       │                                          │
       ▼                                          │
 2. PELENGKAPAN PENDAFTARAN ─────────────────────►│  verifikasi berkas
   (upload dokumen, lengkapi data)                │
       │                                          ▼
       │                                  3. PENILAIAN
       │                                  (input skor, hitung, ranking)
       │                                          │
       ▼                                          ▼
 4. PENGUMUMAN ◄───────────────────────── publish hasil seleksi
   (lihat hasil: Lolos / Tidak Lolos)
```

Status pendaftar mengalir lewat tahapan ini:
`Draft → Menunggu Verifikasi → Terverifikasi → Dinilai → Lolos / Tidak Lolos → (Heregistrasi)`

---

## 1. Modul: PENDAFTARAN

**Referensi PRD:** CM-1, CM-2, AD-2, AD-4 · **Status:** sebagian sudah ada, perlu penyesuaian.

### Tujuan
Calon mahasiswa mengisi biodata awal dan langsung mendapat nomor pendaftaran unik.

### Backend (Laravel)
- [x] Migration `pendaftars` (biodata dasar) — **sudah ada**
- [x] `POST /api/pendaftar` (store) + generate nomor `PMB-2025-XXXX` — **sudah ada**
- [x] `StorePendaftarRequest` validasi field wajib — **sudah ada**
- [ ] Tambah kolom `tahap` (enum alur) dan `tanggal_lahir`, `jenis_kelamin`, `alamat` ke migration
- [ ] Set status awal `Draft` (alih-alih langsung `Menunggu`) agar bisa dibedakan dari yang sudah lengkap

### Frontend (React)
- [x] `FormPendaftaran.jsx` — biodata + submit — **sudah ada**
- [x] Validasi field kosong / format salah — **sudah ada**
- [ ] Setelah submit, arahkan ke halaman pelengkapan (bukan hanya tampilkan nomor)
- [ ] Simpan nomor pendaftaran ke `sessionStorage` untuk lanjut ke tahap berikutnya

### Acceptance Criteria
- Form gagal submit bila ada field wajib kosong / format email/HP salah
- Nomor pendaftaran berformat `PMB-2025-XXXX` dan unik
- Setelah berhasil, user diarahkan ke langkah pelengkapan dengan nomor terbawa

---

## 2. Modul: PELENGKAPAN PENDAFTARAN

**Referensi PRD:** turunan CM-1 (perluasan) · **Status:** BARU.

### Tujuan
Pendaftar melengkapi dokumen & data tambahan; admin memverifikasi kelengkapan sebelum penilaian.

### Backend (Laravel)
- [ ] Migration `dokumen_pendaftar` — relasi `belongsTo` pendaftar
  - kolom: `pendaftar_id`, `jenis_dokumen` (ijazah, foto, rapor, KTP), `path`, `status_verifikasi` (Menunggu/Diterima/Ditolak), `catatan`
- [ ] Konfigurasi `storage` (disk `public`) + `php artisan storage:link`
- [ ] `POST /api/pendaftar/{nomor}/dokumen` (publik, multipart) — upload, validasi `mimes` & `max:2048`
- [ ] `GET /api/pendaftar/{nomor}/dokumen` — daftar dokumen pendaftar
- [ ] `PATCH /api/dokumen/{id}/verifikasi` (admin/Sanctum) — terima/tolak + catatan
- [ ] Saat semua dokumen wajib `Diterima` → ubah status pendaftar jadi `Terverifikasi`
- [ ] `UploadDokumenRequest` untuk validasi file

### Frontend (React)
- [ ] `PelengkapanPendaftaran.jsx` — daftar slot dokumen wajib + komponen upload (drag/drop atau file input)
- [ ] Indikator progres kelengkapan (mis. "3 dari 4 dokumen terunggah")
- [ ] Status per dokumen (Menunggu / Diterima / Ditolak + catatan admin)
- [ ] Di admin: panel verifikasi dokumen per pendaftar (preview, tombol Terima/Tolak)
- [ ] Tambah method `dokumenApi` di `src/utils/api.js`

### Acceptance Criteria
- Hanya file dengan tipe & ukuran valid yang bisa diunggah
- Admin bisa terima/tolak tiap dokumen dengan catatan
- Status pendaftar otomatis naik ke `Terverifikasi` saat dokumen wajib lengkap & diterima

---

## 3. Modul: PENILAIAN

**Referensi PRD:** turunan AD-3 (perluasan seleksi) · **Status:** BARU.

### Tujuan
Admin/penilai memasukkan skor per pendaftar (terverifikasi), sistem menghitung total dan menyusun ranking sebagai dasar kelulusan.

### Backend (Laravel)
- [ ] Migration `penilaians` — relasi `belongsTo` pendaftar
  - kolom: `pendaftar_id`, `nilai_akademik`, `nilai_tes`, `nilai_wawancara` (nullable), `total_nilai`, `dinilai_oleh`, `catatan`
- [ ] Hitung `total_nilai` (mis. bobot terkonfigurasi) saat simpan
- [ ] `GET /api/penilaian` (admin) — daftar pendaftar terverifikasi + skornya, urut by `total_nilai`
- [ ] `POST /api/pendaftar/{id}/penilaian` (admin) — input/update skor
- [ ] Setelah dinilai → status pendaftar jadi `Dinilai`
- [ ] (Opsional) endpoint ranking per prodi dengan kuota

### Frontend (React)
- [ ] `Penilaian.jsx` (admin) — tabel pendaftar terverifikasi, input skor inline/modal
- [ ] Kolom total nilai terhitung otomatis + ranking
- [ ] Filter per prodi/jalur, urut by nilai
- [ ] Hanya pendaftar berstatus `Terverifikasi`/`Dinilai` yang bisa dinilai
- [ ] Tambah method `penilaianApi` di `src/utils/api.js`

### Acceptance Criteria
- Skor hanya bisa diinput untuk pendaftar `Terverifikasi`
- Total nilai konsisten dengan rumus bobot dan tampil real-time
- Ranking akurat dan terurut menurun

---

## 4. Modul: PENGUMUMAN

**Referensi PRD:** CM-3, AD-3, AD-8 · **Status:** sebagian (cek status) ada, publish & notifikasi BARU.

### Tujuan
Admin mempublikasikan hasil seleksi; pendaftar melihat hasil resmi (Lolos / Tidak Lolos) lewat nomor pendaftaran.

### Backend (Laravel)
- [x] `GET /api/pendaftar/{nomor}` (cek status) — **sudah ada**
- [x] `PATCH /api/pendaftar/{id}/status` — **sudah ada**
- [ ] Kolom `diumumkan_at` (timestamp) pada `pendaftars` — hasil hanya tampil setelah dipublikasikan
- [ ] `POST /api/pengumuman/publish` (admin) — tetapkan Lolos/Tidak Lolos berdasarkan ranking/kuota, set `diumumkan_at`
- [ ] Endpoint cek status hanya tampilkan keputusan kelulusan bila `diumumkan_at` terisi (sebelum itu: "sedang diproses")
- [ ] (Opsional AD-8) kirim notifikasi (email/log) ke pendaftar saat diumumkan

### Frontend (React)
- [x] `CekStatus.jsx` — input nomor, tampilkan status — **sudah ada, perlu penyesuaian**
- [ ] Tampilan hasil resmi: kartu "SELAMAT, Anda LOLOS" / "Mohon maaf, belum lolos" — hanya muncul jika sudah diumumkan
- [ ] Bila status `Lolos` + sudah diumumkan → tampilkan tombol Heregistrasi (sudah ada alurnya)
- [ ] Di admin: tombol "Publikasikan Pengumuman" + ringkasan (jumlah lolos/tidak per prodi) sebelum konfirmasi
- [ ] State sebelum pengumuman: tampilkan "Hasil belum diumumkan"

### Acceptance Criteria
- Hasil kelulusan tidak bocor sebelum admin menekan publish
- Setelah publish, cek status menampilkan keputusan yang benar per pendaftar
- Pendaftar lolos dapat melanjutkan ke heregistrasi

---

## 5. Perubahan Skema Data (Ringkasan Migration Baru)

| Tabel | Tipe | Kolom Kunci |
|-------|------|-------------|
| `pendaftars` (alter) | ubah | `+ tahap`, `+ tanggal_lahir`, `+ jenis_kelamin`, `+ alamat`, `+ diumumkan_at` |
| `dokumen_pendaftar` | baru | `pendaftar_id`, `jenis_dokumen`, `path`, `status_verifikasi`, `catatan` |
| `penilaians` | baru | `pendaftar_id`, `nilai_akademik`, `nilai_tes`, `nilai_wawancara`, `total_nilai`, `dinilai_oleh` |

> Catatan: pertahankan konvensi `snake_case` (Laravel default) seperti keputusan teknis di `claude.md`.

---

## 6. Ringkasan Endpoint API

| Method | Endpoint | Auth | Modul | Status |
|--------|----------|------|-------|--------|
| POST | `/api/pendaftar` | publik | Pendaftaran | ada |
| GET | `/api/pendaftar/{nomor}` | publik | Pendaftaran/Pengumuman | ada |
| POST | `/api/pendaftar/{nomor}/dokumen` | publik | Pelengkapan | baru |
| GET | `/api/pendaftar/{nomor}/dokumen` | publik | Pelengkapan | baru |
| PATCH | `/api/dokumen/{id}/verifikasi` | admin | Pelengkapan | baru |
| GET | `/api/penilaian` | admin | Penilaian | baru |
| POST | `/api/pendaftar/{id}/penilaian` | admin | Penilaian | baru |
| POST | `/api/pengumuman/publish` | admin | Pengumuman | baru |
| PATCH | `/api/pendaftar/{id}/status` | admin | Pengumuman | ada |

---

## 7. Urutan Pengerjaan (Milestone)

1. **M1 — Pendaftaran** (penyesuaian): tambah kolom biodata + status `Draft`, redirect ke pelengkapan.
2. **M2 — Pelengkapan**: migration dokumen, upload, verifikasi admin, status `Terverifikasi`.
3. **M3 — Penilaian**: migration penilaian, input skor, ranking, status `Dinilai`.
4. **M4 — Pengumuman**: publish hasil, gating `diumumkan_at`, tampilan hasil + heregistrasi.

Setiap milestone: backend dulu (migration → controller → route → request), lalu frontend (api.js → komponen), lalu uji alur end-to-end.

---

## 8. Risiko & Catatan

- **File upload (prototype):** simpan di disk lokal `public`; belum perlu cloud storage (out of scope PRD).
- **Bobot penilaian:** definisikan di config/constanta agar mudah diubah saat demo.
- **Konsistensi status:** satu sumber kebenaran status di backend; frontend hanya menampilkan.
- **Out of scope** (sesuai PRD §7): pembayaran online, integrasi SIAKAD, mobile app — jangan dikerjakan.
