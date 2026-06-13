# Development Plan — Sistem PMB Kampus

| Field | Keterangan |
|---|---|
| Dokumen | Development Plan (Devplan) |
| Versi | 1.0 |
| Tanggal | 13 Juni 2026 |
| Referensi | `prd.md` |

Dokumen ini menjelaskan **alur pengembangan**, **technology stack**, dan **struktur proyek** untuk membangun Sistem PMB Kampus. Pengembangan dibagi 3 fase (FE landing/pendaftaran → FE admin → integrasi BE) sesuai PRD.

---

## 1. Technology Stack

### 1.1 Frontend
| Komponen | Pilihan | Catatan |
|---|---|---|
| Framework | React 18 + Vite | SPA, build cepat |
| Bahasa | TypeScript | Type-safe |
| UI Library | shadcn/ui | Komponen di-generate ke dalam repo |
| Styling | Tailwind CSS | Utility-first |
| Routing | React Router | Routing client-side |
| Data Fetching | TanStack Query (React Query) | Cache & state server |
| HTTP Client | Axios | Interceptor untuk token |
| Form & Validasi | React Hook Form + Zod | Validasi schema |
| State (UI) | Zustand | State global ringan (auth, dll) |
| Charts | Recharts | Grafik dashboard admin |
| Tabel | TanStack Table | Tabel data admin (sort, filter, paginate) |
| Pembayaran | Midtrans Snap.js | Embed Snap di sisi klien |
| PDF Kartu Peserta | `@react-pdf/renderer` atau render server-side | Lihat bagian 5.3 |

### 1.2 Backend
| Komponen | Pilihan | Catatan |
|---|---|---|
| Framework | Laravel (versi stabil terbaru) | REST API |
| Bahasa | PHP 8.2+ | |
| Auth | Laravel Sanctum | Token-based untuk peserta & admin |
| Database | Supabase (PostgreSQL) | Koneksi via driver `pgsql` |
| Storage | Supabase Storage | Dokumen pendukung & aset |
| Pembayaran | Midtrans PHP SDK | Create transaksi + handle webhook |
| WhatsApp | Fonnte API (HTTP) | Via Laravel HTTP Client |
| PDF (opsional) | `barryvdh/laravel-dompdf` | Generate kartu peserta server-side |
| Queue | Laravel Queue (database/redis) | Kirim WA & proses async |

### 1.3 Tooling & Infra
| Komponen | Pilihan |
|---|---|
| Package Manager FE | pnpm / npm |
| Linting & Format | ESLint + Prettier (FE), Laravel Pint (BE) |
| Env Management | `.env` (FE: `VITE_*`, BE: Laravel env) |
| Version Control | Git (mono-repo: folder `frontend` & `backend`) |
| API Testing | Postman / Thunder Client |

> Catatan versi: ikuti versi stabil terbaru saat implementasi. Gunakan Context7/dokumentasi resmi untuk verifikasi sintaks API terkini sebelum coding.

---

## 2. Arsitektur Sistem

```
┌─────────────────┐        REST API (JSON)        ┌──────────────────┐
│   Frontend      │  ───────────────────────────► │   Backend         │
│  React + shadcn │  ◄─────────────────────────── │   Laravel API     │
└─────────────────┘        Sanctum token          └────────┬─────────┘
        │                                                   │
        │ Snap.js                                           │ pgsql
        ▼                                                   ▼
┌─────────────────┐                               ┌──────────────────┐
│  Midtrans Snap  │ ◄── webhook ──────────────────│  Supabase        │
└─────────────────┘                               │  (PostgreSQL +   │
┌─────────────────┐                               │   Storage)       │
│  Fonnte (WA)    │ ◄── HTTP (server-side) ────── │                  │
└─────────────────┘                               └──────────────────┘
```

- **Frontend** murni mengonsumsi REST API; tidak mengakses Supabase langsung (semua lewat Laravel) untuk keamanan & konsistensi.
- **Webhook** Midtrans dan **pengiriman WA** ditangani backend.
- **Upload file** dilakukan via endpoint backend → Supabase Storage (validasi tipe & ukuran di server).

---

## 3. Struktur Proyek (Mono-repo)

```
pmb-siakad/
├── prd.md
├── devplan.md
├── frontend/                      # React + Vite + shadcn/ui
│   ├── public/
│   ├── src/
│   │   ├── main.tsx
│   │   ├── App.tsx
│   │   ├── routes/                # Definisi route & layout
│   │   │   ├── index.tsx
│   │   │   └── ProtectedRoute.tsx
│   │   ├── layouts/
│   │   │   ├── PublicLayout.tsx
│   │   │   ├── PesertaLayout.tsx
│   │   │   └── AdminLayout.tsx
│   │   ├── pages/
│   │   │   ├── public/            # Landing, jadwal, form daftar, login
│   │   │   │   ├── LandingPage.tsx
│   │   │   │   ├── JadwalDetailPage.tsx
│   │   │   │   ├── PendaftaranPage.tsx
│   │   │   │   ├── PembayaranPage.tsx
│   │   │   │   └── LoginPage.tsx
│   │   │   ├── peserta/           # Dashboard peserta
│   │   │   │   ├── StatusPage.tsx
│   │   │   │   ├── EditDataPage.tsx
│   │   │   │   ├── DokumenPage.tsx
│   │   │   │   └── KartuPesertaPage.tsx
│   │   │   └── admin/             # Panel admin
│   │   │       ├── DashboardPage.tsx
│   │   │       ├── PendaftarPage.tsx
│   │   │       ├── PembayaranPage.tsx
│   │   │       ├── CarouselPage.tsx
│   │   │       ├── ProdiPage.tsx
│   │   │       ├── JalurPage.tsx
│   │   │       ├── JadwalPage.tsx
│   │   │       ├── UserPage.tsx
│   │   │       └── NotifikasiPage.tsx
│   │   ├── components/
│   │   │   ├── ui/                # Komponen shadcn (generated)
│   │   │   ├── common/            # Komponen reusable (DataTable, dll)
│   │   │   └── features/          # Komponen per fitur (Carousel, dll)
│   │   ├── lib/
│   │   │   ├── api.ts             # Instance Axios + interceptor
│   │   │   ├── queryClient.ts
│   │   │   └── utils.ts
│   │   ├── hooks/                 # Custom hooks (useAuth, useJadwal, ...)
│   │   ├── services/             # API service per domain
│   │   ├── store/                # Zustand stores
│   │   ├── types/                # Tipe TypeScript (DTO)
│   │   └── styles/
│   ├── .env                      # VITE_API_URL, VITE_MIDTRANS_CLIENT_KEY
│   ├── components.json           # Konfigurasi shadcn
│   ├── tailwind.config.ts
│   ├── vite.config.ts
│   └── package.json
│
└── backend/                      # Laravel API
    ├── app/
    │   ├── Http/
    │   │   ├── Controllers/Api/
    │   │   │   ├── Auth/
    │   │   │   ├── PendaftaranController.php
    │   │   │   ├── PembayaranController.php
    │   │   │   ├── DokumenController.php
    │   │   │   ├── KartuPesertaController.php
    │   │   │   └── Admin/        # Controller area admin
    │   │   ├── Requests/         # Form Request (validasi)
    │   │   ├── Resources/        # API Resources (transform JSON)
    │   │   └── Middleware/
    │   ├── Models/               # User, Pendaftar, Jalur, Jadwal, dll
    │   ├── Services/             # MidtransService, FonnteService, dll
    │   └── Jobs/                 # KirimWhatsappJob, dll
    ├── config/                   # midtrans.php, fonnte.php, services.php
    ├── database/
    │   ├── migrations/
    │   ├── seeders/
    │   └── factories/
    ├── routes/
    │   └── api.php
    ├── storage/
    ├── .env                      # DB Supabase, Midtrans, Fonnte keys
    └── composer.json
```

---

## 4. Alur Pengembangan (Development Flow)

Strategi: bangun **frontend lebih dulu dengan data mock**, lalu sambungkan ke backend di Fase 3. Ini memungkinkan validasi UX lebih awal.

### Fase 0 — Persiapan
1. Inisialisasi mono-repo & Git.
2. Setup `frontend/` (Vite + React + TS + Tailwind + shadcn/ui).
3. Siapkan struktur folder, routing dasar, dan layout (public/peserta/admin).
4. Buat mock API layer (data dummy) agar UI bisa dikembangkan tanpa backend.

### Fase 1 — Landing & Pendaftaran (Frontend)
1. Landing page: carousel, info kampus, daftar prodi.
2. Komponen jadwal PMB: list, halaman detail, tombol unduh brosur.
3. Form pendaftaran (nama, NIK, no HP, email, jenis kelamin, prodi tujuan); jalur otomatis dari jadwal aktif.
4. Validasi form (React Hook Form + Zod), termasuk cek duplikasi NIK (mock).
5. Halaman pembayaran (UI Snap, kondisional jika berbayar).
6. Halaman login peserta (nomor pendaftaran + password).
7. Dashboard peserta: status, ubah data, unggah dokumen (mengikuti persyaratan dinamis), detail pembayaran, unduh kartu peserta.
8. Polish responsif & UX.

### Fase 2 — Admin (Frontend)
1. Layout admin + auth guard + sidebar.
2. Dashboard: grafik (Recharts) & kartu ringkasan.
3. DataTable reusable: pendaftar & pembayaran (filter, search, paginate, detail).
4. CRUD: carousel, prodi, jalur (+ konfigurasi persyaratan dokumen), jadwal, user.
5. Aksi status pendaftaran (loloskan/tolak) + form kirim notifikasi WA.
6. Unduh kartu peserta per pendaftar.

### Fase 3 — Integrasi Backend
1. Setup Laravel + koneksi Supabase (PostgreSQL).
2. Migrasi, model, relasi, & seeder (sesuai model data PRD bagian 4).
3. Auth Sanctum (peserta & admin) + middleware role.
4. Endpoint publik: carousel, jadwal, prodi, pendaftaran, pembayaran.
5. Integrasi Midtrans (create Snap token + webhook) → update status pembayaran.
6. Integrasi Fonnte via Service + Queue (nomor pendaftaran & password, perubahan status).
7. Upload dokumen → Supabase Storage (validasi 10MB, tipe sesuai persyaratan).
8. Endpoint admin (dashboard summary, CRUD, ubah status, kirim WA).
9. Generate kartu peserta (PDF).
10. Ganti mock layer di frontend dengan API nyata + uji end-to-end.

---

## 5. Catatan Teknis Penting

### 5.1 Autentikasi
- Sanctum token disimpan aman di frontend; Axios interceptor menyisipkan `Authorization: Bearer`.
- Middleware role memisahkan akses `peserta` vs `admin`.

### 5.2 Unggah Dokumen
- Validasi di server: ukuran ≤ 10 MB dan MIME sesuai `persyaratan_dokumen.jenis_file`.
- Simpan ke Supabase Storage; simpan metadata (`file_url`, `ukuran`, `mime_type`) di tabel `dokumen`.
- Cek kelengkapan dokumen wajib sebelum status "Verifikasi Dokumen".

### 5.3 Kartu Peserta (PDF)
- **Opsi A (server-side, direkomendasikan):** Laravel + DomPDF, endpoint `GET /kartu-peserta/{id}` mengembalikan PDF. Otorisasi: peserta hanya kartunya sendiri, admin semua.
- **Opsi B (client-side):** `@react-pdf/renderer` di frontend.

### 5.4 Webhook Midtrans
- Endpoint publik tanpa Sanctum, diverifikasi via **signature key**.
- Update `pembayaran.status`; jika sukses → trigger WA & lanjut status pendaftaran.

### 5.5 Notifikasi WA (Fonnte)
- Dibungkus `FonnteService` dan dijalankan via Queue/Job agar tidak memblok request.
- Setiap pengiriman dicatat di `notifikasi_log` (status kirim, retry bila gagal).

### 5.6 Environment Variables
**Frontend (`.env`)**
```
VITE_API_URL=
VITE_MIDTRANS_CLIENT_KEY=
```
**Backend (`.env`)**
```
DB_CONNECTION=pgsql
DB_HOST=            # Supabase host
DB_PORT=5432
DB_DATABASE=
DB_USERNAME=
DB_PASSWORD=
SUPABASE_URL=
SUPABASE_STORAGE_BUCKET=
MIDTRANS_SERVER_KEY=
MIDTRANS_CLIENT_KEY=
MIDTRANS_IS_PRODUCTION=false
FONNTE_TOKEN=
```

---

## 6. Daftar Endpoint API (Rancangan Awal)

| Method | Endpoint | Akses | Fungsi |
|---|---|---|---|
| GET | `/api/carousel` | Publik | List carousel aktif |
| GET | `/api/jadwal` | Publik | List jadwal PMB |
| GET | `/api/jadwal/{id}` | Publik | Detail jadwal + brosur |
| GET | `/api/prodi` | Publik | List program studi |
| GET | `/api/jalur/aktif` | Publik | Jalur dari jadwal yang dibuka |
| POST | `/api/pendaftaran` | Publik | Submit pendaftaran (validasi NIK unik) |
| POST | `/api/pembayaran/snap` | Publik/Peserta | Buat Snap token |
| POST | `/api/pembayaran/webhook` | Midtrans | Webhook status pembayaran |
| POST | `/api/auth/login` | Publik | Login peserta/admin |
| GET | `/api/peserta/status` | Peserta | Status pendaftaran |
| PUT | `/api/peserta/data` | Peserta | Ubah data |
| POST | `/api/peserta/dokumen` | Peserta | Unggah dokumen |
| GET | `/api/kartu-peserta/{id}` | Peserta/Admin | Unduh kartu (PDF) |
| GET | `/api/admin/dashboard` | Admin | Summary & grafik |
| GET | `/api/admin/pendaftar` | Admin | List pendaftar |
| PUT | `/api/admin/pendaftar/{id}/status` | Admin | Ubah/loloskan/tolak |
| GET | `/api/admin/pembayaran` | Admin | List & validasi |
| POST | `/api/admin/notifikasi` | Admin | Kirim WA manual |
| CRUD | `/api/admin/{carousel\|prodi\|jalur\|jadwal\|user}` | Admin | Data master |

> Daftar di atas adalah rancangan awal dan dapat disesuaikan saat implementasi Fase 3.

---

## 7. Definition of Done (per Fase)
- **Fase 1 & 2:** seluruh halaman tampil benar dengan data mock, responsif, lolos lint, dan navigasi/alur sesuai PRD.
- **Fase 3:** seluruh endpoint berfungsi, integrasi Midtrans & Fonnte teruji (sandbox), upload & PDF berjalan, dan frontend tersambung penuh ke API (uji end-to-end alur pendaftaran → bayar → notifikasi → status).
