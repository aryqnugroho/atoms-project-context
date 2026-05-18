# Panduan Menjalankan ATOMS-Maintenance

> **Dokumen ini ditulis untuk developer, tim IT, dan siapa pun yang perlu menjalankan, mengembangkan, atau men-deploy sistem ATOMS-Maintenance.**

---

## Daftar Isi

1. [Tujuan Dokumen](#1-tujuan-dokumen)
2. [Ringkasan Arsitektur](#2-ringkasan-arsitektur)
3. [Struktur Folder Penting](#3-struktur-folder-penting)
4. [Aturan Port Lokal](#4-aturan-port-lokal)
5. [Prasyarat](#5-prasyarat)
6. [Setup Database Lokal](#6-setup-database-lokal)
7. [Menjalankan Backend atoms-rostering](#7-menjalankan-backend-atoms-rostering)
8. [Menjalankan Frontend atoms-rostering](#8-menjalankan-frontend-atoms-rostering)
9. [Menjalankan Backend atoms-maintenance](#9-menjalankan-backend-atoms-maintenance)
10. [Menjalankan Frontend atoms-maintenance](#10-menjalankan-frontend-atoms-maintenance)
11. [Alur Integrasi SSO](#11-alur-integrasi-sso)
12. [Bagian atoms-rostering yang Diubah](#12-bagian-atoms-rostering-yang-diubah)
13. [Validasi Integrasi Lokal](#13-validasi-integrasi-lokal)
14. [Troubleshooting Umum](#14-troubleshooting-umum)
15. [Catatan Deployment Server Kantor](#15-catatan-deployment-server-kantor)
16. [Urutan Singkat Menjalankan Semua Service Lokal](#16-urutan-singkat-menjalankan-semua-service-lokal)
17. [Checklist Akhir](#17-checklist-akhir)

---

## 1. Tujuan Dokumen

Dokumen ini berisi:

- Cara menjalankan **atoms-maintenance** dari nol hingga siap dipakai.
- Cara menjalankan **backend (Laravel)** dan **frontend (React/Vite)** masing-masing.
- Cara **mengintegrasikan atoms-maintenance dengan atoms-rostering** (Single Sign-On / SSO).
- Aturan port agar backend dan frontend tidak saling bertabrakan.
- Perubahan **minimum** yang diperlukan di atoms-rostering agar menu Maintenance bisa mengarah ke atoms-maintenance.
- Catatan persiapan **deployment ke server kantor**.

---

## 2. Ringkasan Arsitektur

```
ATOMS PROJECT
├── atoms-rostering   ← Aplikasi login, roster, dan data kepegawaian
│                        (sumber kebenaran untuk autentikasi)
└── atoms-maintenance ← Aplikasi manajemen maintenance operasional
                         (menggunakan token dari atoms-rostering)
```

### Prinsip Utama

| Prinsip | Penjelasan |
|---|---|
| **atoms-rostering** adalah sumber login | Semua user login melalui atoms-rostering |
| **atoms-maintenance tidak punya login sendiri** | User diarahkan ke rostering jika belum login |
| **Autentikasi via Sanctum token** | Token opaque Laravel Sanctum, bukan JWT |
| **Validasi token via API** | Backend maintenance memanggil `/api/auth/me` rostering untuk validasi |
| **db_rostering read-only** | atoms-maintenance boleh membaca db_rostering, tapi tidak boleh menulis |
| **Token disimpan di sessionStorage** | Bukan localStorage; session berakhir saat tab ditutup |

### Alur Login SSO

```
User Browser
    │
    ▼
atoms-rostering (Login Page)
    │  User isi email + password
    │  Sanctum mengeluarkan opaque token
    ▼
Dashboard atoms-rostering
    │  User klik card "Maintenance"
    │  MenuGrid.tsx mengambil token dari sessionStorage
    ▼
Redirect ke atoms-maintenance frontend
    URL: http://localhost:5173?token={sanctum_token}
    │
    ▼
atoms-maintenance frontend (AuthContext)
    │  Baca ?token dari URL
    │  Simpan ke sessionStorage['auth_token']
    │  Hapus token dari URL (history.replaceState)
    │  Kirim POST ke backend maintenance untuk verifikasi
    ▼
atoms-maintenance backend (GET /api/v1/auth/verify)
    │  Forward ke atoms-rostering: GET /api/auth/me
    │  Header: Authorization: Bearer {token}
    ▼
atoms-rostering backend
    │  Validasi token Sanctum
    │  Return data user jika valid
    ▼
atoms-maintenance frontend
    │  Token valid → user masuk dashboard
    │  Token invalid → redirect ke login rostering
```

---

## 3. Struktur Folder Penting

```
ATOMS-PROJECT/
│
├── atoms-rostering/
│   ├── backend_atoms/          ← Backend Laravel atoms-rostering
│   │   ├── app/
│   │   ├── routes/api.php
│   │   ├── .env
│   │   └── .env.example
│   └── frontend_atoms/         ← Frontend React atoms-rostering
│       ├── src/
│       │   └── components/feature/home/MenuGrid.tsx  ← File yang diubah untuk SSO
│       ├── .env
│       └── package.json
│
├── atoms-maintenance/
│   ├── backend_atoms-maintenance/   ← Backend Laravel atoms-maintenance
│   │   ├── app/
│   │   │   ├── Http/
│   │   │   │   ├── Controllers/Api/V1/WorkOrder/
│   │   │   │   └── Middleware/
│   │   │   │       ├── MockAuthMiddleware.php
│   │   │   │       └── VerifyRosteringToken.php
│   │   │   └── Services/
│   │   │       ├── RosteringAuthService.php
│   │   │       └── LocalUserResolver.php
│   │   ├── routes/api.php
│   │   ├── .env                     ← Berisi konfigurasi DB + ROSTERING_API_URL
│   │   └── .env.example             ← Template env untuk developer baru
│   └── frontend_atoms-maintenance/  ← Frontend React atoms-maintenance
│       ├── src/
│       │   └── contexts/AuthContext.tsx  ← Menangani token dari URL
│       ├── .env                     ← VITE_API_URL + VITE_ROSTERING_FRONTEND_URL
│       └── package.json
│
├── .agents/                    ← Dokumentasi teknis untuk AI coding assistant
├── .kiro/                      ← Steering rules dan skill definitions
└── CARA_MENJALANKAN_ATOMS_MAINTENANCE.md  ← File ini
```

---

## 4. Aturan Port Lokal

**Port yang digunakan project ini (sesuai konfigurasi aktual):**

| Aplikasi | Layer | Port | URL Lokal |
|---|---|---|---|
| atoms-rostering | Backend Laravel | **8001** | `http://localhost:8001` |
| atoms-rostering | Frontend React/Vite | **5174** | `http://localhost:5174` |
| atoms-maintenance | Backend Laravel | **8000** | `http://localhost:8000` |
| atoms-maintenance | Frontend React/Vite | **5173** | `http://localhost:5173` |

> **Catatan penting:**
> - atoms-rostering frontend harus dijalankan dengan `--port 5174` secara eksplisit karena Vite default adalah 5173 (sama dengan atoms-maintenance frontend).
> - Jangan menjalankan dua backend Laravel pada port yang sama.
> - Jangan menjalankan dua frontend Vite pada port yang sama tanpa `--port` eksplisit.
> - Pastikan `VITE_API_URL` di frontend maintenance mengarah ke **port 8000** (backend maintenance), bukan ke 8001.
> - Pastikan `ROSTERING_API_URL` di backend maintenance mengarah ke **port 8001** (backend rostering).

---

## 5. Prasyarat

Pastikan tool berikut sudah terinstall sebelum menjalankan project:

| Tool | Keterangan |
|---|---|
| **PHP** | Sesuaikan dengan `composer.json` di backend (Laravel 13.x menggunakan PHP ≥ 8.2) |
| **Composer** | Dependency manager PHP |
| **Node.js** | Sesuaikan dengan `package.json` di frontend (Vite 8.x umumnya butuh Node.js ≥ 18) |
| **npm** | Dependency manager JavaScript (sudah termasuk bersama Node.js) |
| **PostgreSQL** | Database untuk atoms-maintenance dan atoms-rostering |
| **Git** | Version control |
| **RTK** *(opsional tapi direkomendasikan)* | CLI proxy untuk efisiensi terminal ([github.com/rtk-ai/rtk](https://github.com/rtk-ai/rtk)) |

> Untuk menginstall RTK di Windows: unduh `rtk-x86_64-pc-windows-msvc.zip` dari halaman Releases di GitHub, extract, dan tambahkan `rtk.exe` ke PATH.

---

## 6. Setup Database Lokal

### Database yang diperlukan

| Nama Database | Dipakai Oleh | Akses dari atoms-maintenance |
|---|---|---|
| `atoms_rostering` | atoms-rostering | **Read-only** — hanya baca data roster dan user |
| `atoms_maintenance` | atoms-maintenance | **Read-write** — semua data maintenance disimpan di sini |

> Nama database mengikuti konfigurasi di file `.env` masing-masing project. Nama di atas adalah nilai default.

### Langkah setup

1. Buka psql atau pgAdmin.
2. Buat dua database:

```sql
CREATE DATABASE atoms_rostering;
CREATE DATABASE atoms_maintenance;
```

3. Pastikan user PostgreSQL yang digunakan memiliki akses ke kedua database.

> **Aturan penting:** atoms-maintenance **tidak boleh** menulis ke `atoms_rostering`. Koneksi `rostering` di `config/database.php` atoms-maintenance bersifat read-only.

---

## 7. Menjalankan Backend atoms-rostering

Backend atoms-rostering adalah **sumber autentikasi**. Harus dijalankan **pertama kali** sebelum backend maintenance.

```bash
# Masuk ke folder backend atoms-rostering
cd atoms-rostering/backend_atoms

# Install dependency PHP
rtk composer install

# Salin file env jika belum ada
cp .env.example .env

# Atur konfigurasi database di .env:
# DB_DATABASE=atoms_rostering
# DB_USERNAME=postgres
# DB_PASSWORD=password_anda

# Generate app key
rtk php artisan key:generate

# Jalankan migrasi database
rtk php artisan migrate

# Jalankan seeder jika diperlukan oleh rostering
# (ikuti dokumentasi atoms-rostering tersendiri)
# rtk php artisan db:seed

# Jalankan Laravel server di port 8001
rtk php artisan serve --host=127.0.0.1 --port=8001
```

> **Endpoint penting:**
> - `POST /api/auth/login` — Login user, mengembalikan Sanctum token
> - `GET /api/auth/me` — Validasi token dan mengembalikan data user
>
> Kedua endpoint ini wajib berjalan agar SSO atoms-maintenance berfungsi.

---

## 8. Menjalankan Frontend atoms-rostering

```bash
# Masuk ke folder frontend atoms-rostering
cd atoms-rostering/frontend_atoms

# Install dependency JavaScript
rtk npm install

# Salin file env jika belum ada (sesuaikan dengan file .env.example yang ada)
# Pastikan variabel VITE_API_URL atau yang setara mengarah ke backend rostering port 8001
cp .env.example .env
# Edit .env: sesuaikan VITE_API_URL=http://127.0.0.1:8001

# Jalankan dev server di port 5174
# (port 5174 wajib eksplisit agar tidak bentrok dengan frontend maintenance di 5173)
rtk npm run dev -- --host 127.0.0.1 --port 5174
```

> **Catatan:** Frontend rostering menggunakan **Vite** yang default port-nya 5173. Karena frontend maintenance juga default 5173, rostering **wajib** dijalankan dengan `--port 5174`.

---

## 9. Menjalankan Backend atoms-maintenance

```bash
# Masuk ke folder backend atoms-maintenance
cd atoms-maintenance/backend_atoms-maintenance

# Install dependency PHP
rtk composer install

# Salin file env jika belum ada
cp .env.example .env
```

### Konfigurasi .env yang wajib diisi

Buka file `.env` dan pastikan nilai berikut benar:

```dotenv
# URL aplikasi ini
APP_URL=http://localhost:8000

# Database maintenance (read-write)
DB_CONNECTION=pgsql
DB_HOST=localhost
DB_PORT=5432
DB_DATABASE=atoms_maintenance
DB_USERNAME=postgres
DB_PASSWORD=password_anda

# Database rostering (read-only — untuk membaca data roster/personnel)
ROSTERING_DB_HOST=localhost
ROSTERING_DB_PORT=5432
ROSTERING_DB_DATABASE=atoms_rostering
ROSTERING_DB_USERNAME=postgres
ROSTERING_DB_PASSWORD=password_anda

# URL backend rostering — digunakan untuk validasi Sanctum token
ROSTERING_API_URL=http://localhost:8001

# URL frontend rostering — digunakan untuk redirect logout
ROSTERING_FRONTEND_URL=http://localhost:5174

# Mode autentikasi:
# false = SSO mode (production/staging) — validasi token ke rostering
# true  = mock mode (development tanpa rostering) — gunakan mock-token
DEV_MOCK_AUTH=false

# Domain untuk Sanctum stateful (sesuaikan dengan port frontend maintenance)
SANCTUM_STATEFUL_DOMAINS=localhost:5173
SESSION_DOMAIN=localhost
```

```bash
# Generate app key
rtk php artisan key:generate

# Jalankan migrasi database maintenance
# (JANGAN jalankan db:seed default — WorkOrderSeeder tidak diperlukan)
rtk php artisan migrate

# Sinkronisasi data user dari atoms-rostering ke local_users atoms-maintenance
# (Opsional tapi direkomendasikan setelah database kosong)
rtk php artisan local-users:sync

# Jalankan Laravel server di port 8000
rtk php artisan serve --host=127.0.0.1 --port=8000
```

> **Catatan penting:**
> - **Jangan** jalankan `php artisan db:seed` secara default. `WorkOrderSeeder` dan `MockUserSeeder` sudah ditandai deprecated dan tidak boleh dipanggil.
> - Database `atoms_maintenance` dimulai **kosong** (tidak ada Work Order dummy).
> - `php artisan local-users:sync` akan menarik data user aktif dari `atoms_rostering` secara read-only dan menyimpannya di `local_users` maintenance.

---

## 10. Menjalankan Frontend atoms-maintenance

```bash
# Masuk ke folder frontend atoms-maintenance
cd atoms-maintenance/frontend_atoms-maintenance

# Install dependency JavaScript
rtk npm install
```

### Konfigurasi .env frontend

Buka file `.env` dan pastikan nilai berikut benar:

```dotenv
# URL backend atoms-maintenance
VITE_API_URL=http://localhost:8000/api

# Mode autentikasi:
# false = SSO mode (production) — token dari atoms-rostering
# true  = mock mode (development tanpa rostering)
VITE_DEV_MOCK_AUTH=false

# URL frontend atoms-rostering — untuk redirect saat user belum login
VITE_ROSTERING_FRONTEND_URL=http://localhost:5174
```

```bash
# Jalankan dev server di port 5173 (Vite default)
rtk npm run dev -- --host 127.0.0.1 --port 5173
```

> **Catatan:** Jika `VITE_DEV_MOCK_AUTH=false`, frontend maintenance akan **langsung redirect ke `VITE_ROSTERING_FRONTEND_URL/login`** jika tidak ada token. Pastikan frontend rostering sudah berjalan di port 5174 sebelum membuka frontend maintenance.

---

## 11. Alur Integrasi SSO

Berikut adalah alur lengkap dari login hingga masuk ke atoms-maintenance:

### Langkah demi langkah

1. **User membuka atoms-rostering** di browser:
   ```
   http://localhost:5174
   ```

2. **User login** dengan email dan password. atoms-rostering mengeluarkan **Laravel Sanctum opaque token** (bukan JWT). Token disimpan di `sessionStorage['auth_token']` browser.

3. **User klik card "Maintenance"** di dashboard atoms-rostering.

4. **MenuGrid.tsx** mengambil token dari sessionStorage dan melakukan redirect:
   ```
   http://localhost:5173?token={sanctum_token_url_encoded}
   ```

5. **Frontend atoms-maintenance** (AuthContext.tsx) memproses redirect:
   - Membaca `?token` dari URL
   - Menyimpan token ke `sessionStorage['auth_token']`
   - Menghapus token dari URL (history.replaceState — agar tidak terlihat di browser history)
   - Mengirim request verifikasi ke backend maintenance

6. **Backend atoms-maintenance** menerima request verifikasi:
   ```
   GET /api/v1/auth/verify
   Authorization: Bearer {sanctum_token}
   ```

7. **Backend atoms-maintenance mem-forward** ke backend atoms-rostering:
   ```
   GET http://localhost:8001/api/auth/me
   Authorization: Bearer {sanctum_token}
   ```

8. **Backend atoms-rostering** memvalidasi token Sanctum dan mengembalikan data user.

9. **Backend atoms-maintenance** menyimpan data user ke `local_users` (auto-sync) dan mengembalikan respons ke frontend.

10. **Frontend atoms-maintenance** menyimpan data user, menghapus URL token, dan menampilkan dashboard maintenance.

11. **Untuk request selanjutnya** (Work Order, roster, dll.), frontend maintenance menyertakan header:
    ```
    Authorization: Bearer {sanctum_token}
    ```
    Backend maintenance memvalidasi setiap request ke rostering.

### Aturan penyimpanan token

| Storage | Dipakai | Tidak Dipakai |
|---|---|---|
| `sessionStorage` | ✅ Token Sanctum | |
| `localStorage` | | ❌ Dilarang |
| Cookie | | ❌ Tidak digunakan untuk token |

---

## 12. Bagian atoms-rostering yang Diubah

> **Aturan penting:** atoms-rostering adalah **READ-ONLY** kecuali satu file berikut.

### File yang dimodifikasi

**File:** `atoms-rostering/frontend_atoms/src/components/feature/home/MenuGrid.tsx`

### Apa yang diubah

Handler klik pada card **"Maintenance"** diubah agar:

1. Mengambil token Sanctum dari sessionStorage menggunakan fungsi `getStoredToken()` dari `authStorage.ts`.
2. Melakukan `window.location.href` ke URL frontend atoms-maintenance disertai token.

### Kode aktual (MenuGrid.tsx)

```tsx
if (item.title === 'Maintenance') {
  const token = getStoredToken();  // ambil dari sessionStorage
  const baseUrl = 'http://localhost:5173';  // URL frontend maintenance
  const url = token
    ? `${baseUrl}?token=${encodeURIComponent(token)}`
    : baseUrl;
  window.location.href = url;
}
```

### Untuk deployment/penyesuaian

Saat deploy ke server, ganti `http://localhost:5173` dengan URL production frontend atoms-maintenance. Idealnya baca dari variabel environment:

```tsx
// Konsep (sesuaikan dengan environment setup atoms-rostering)
const maintenanceUrl = import.meta.env.VITE_MAINTENANCE_FRONTEND_URL || 'http://localhost:5173';
const url = token
  ? `${maintenanceUrl}?token=${encodeURIComponent(token)}`
  : maintenanceUrl;
window.location.href = url;
```

### Yang TIDAK boleh diubah di atoms-rostering

- ❌ Logika login / autentikasi
- ❌ Sistem token Sanctum
- ❌ Database dan migrasi rostering
- ❌ Controller, Model, Service lainnya
- ❌ File lain selain `MenuGrid.tsx`
- ❌ Jangan query `personal_access_tokens` secara manual
- ❌ Jangan buat JWT

---

## 13. Validasi Integrasi Lokal

Setelah semua service berjalan, lakukan pengujian berikut:

### Checklist pengujian

```
[ ] 1. Backend rostering aktif di port 8001
       Test: curl http://localhost:8001/up → harus 200 OK

[ ] 2. Frontend rostering aktif di port 5174
       Buka browser: http://localhost:5174 → harus muncul halaman login

[ ] 3. Backend maintenance aktif di port 8000
       Test: curl http://localhost:8000/up → harus 200 OK

[ ] 4. Frontend maintenance aktif di port 5173
       (Belum bisa dibuka langsung karena akan redirect ke rostering)

[ ] 5. Login ke atoms-rostering
       Masukkan email dan password yang valid

[ ] 6. Klik card "Maintenance" di dashboard rostering
       Harus redirect ke http://localhost:5173?token=...
       lalu URL berubah menjadi http://localhost:5173/dashboard

[ ] 7. Masuk ke dashboard atoms-maintenance tanpa login ulang

[ ] 8. Buka DevTools → tab Network
       Cek request ke http://localhost:8000/api/...
       Harus ada header: Authorization: Bearer {token}

[ ] 9. Refresh halaman atoms-maintenance
       Harus tetap login (token ada di sessionStorage)

[10] 10. Tutup tab lalu buka lagi http://localhost:5173
         Harus redirect ke login rostering (sessionStorage hilang)
```

### Test via curl (opsional)

```bash
# 1. Login ke rostering dan ambil token
curl -X POST http://localhost:8001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@airnav.com","password":"password"}'
# Catat access_token dari respons

# 2. Validasi token via endpoint rostering
curl http://localhost:8001/api/auth/me \
  -H "Authorization: Bearer {TOKEN}"
# Harus mengembalikan data user

# 3. Validasi token via endpoint maintenance
curl http://localhost:8000/api/v1/auth/verify \
  -H "Authorization: Bearer {TOKEN}"
# Harus mengembalikan user valid dari maintenance

# 4. Cek Work Order API
curl http://localhost:8000/api/v1/work-orders \
  -H "Authorization: Bearer {TOKEN}"
# Harus mengembalikan list Work Order (kosong atau berisi data)
```

---

## 14. Troubleshooting Umum

### Masalah: Klik Maintenance kembali ke halaman login rostering

**Kemungkinan penyebab dan solusi:**

| Penyebab | Solusi |
|---|---|
| Token tidak ada di sessionStorage rostering | Coba logout dari rostering dan login ulang |
| Frontend maintenance tidak membaca `?token` dari URL | Cek `AuthContext.tsx` — pastikan kode membaca `new URLSearchParams(window.location.search).get('token')` |
| `ROSTERING_API_URL` salah di backend maintenance | Cek `.env` backend maintenance: `ROSTERING_API_URL=http://localhost:8001` |
| Endpoint `/api/auth/me` rostering tidak jalan | Test manual: `curl http://localhost:8001/api/auth/me -H "Authorization: Bearer TOKEN"` |
| Backend maintenance mengembalikan 401 | Cek log Laravel: `storage/logs/laravel.log` |
| `VITE_DEV_MOCK_AUTH=true` di frontend maintenance | Ubah ke `false` untuk mode SSO real |

---

### Masalah: CORS error di browser

**Solusi:**

1. Cek file `config/cors.php` di backend maintenance:
   ```php
   'allowed_origins' => ['http://localhost:5173'],
   ```
2. Pastikan `SANCTUM_STATEFUL_DOMAINS=localhost:5173` ada di `.env` backend maintenance.
3. Pastikan `APP_URL` di `.env` sesuai dengan port yang dijalankan.
4. Restart backend setelah mengubah `.env`:
   ```bash
   rtk php artisan config:clear
   rtk php artisan serve --host=127.0.0.1 --port=8000
   ```

---

### Masalah: Port sudah digunakan

**Solusi:**

Windows CMD — cek proses yang memakai port tertentu:
```cmd
netstat -ano | findstr :8000
taskkill /PID {nomor_PID} /F
```

PowerShell:
```powershell
Get-Process -Id (Get-NetTCPConnection -LocalPort 8000).OwningProcess
```

Kemudian restart service dengan port yang berbeda atau hentikan proses yang bentrok.

---

### Masalah: Work Order API mengembalikan 401 Unauthorized

**Langkah debugging:**

1. Cek apakah request menyertakan header `Authorization: Bearer {token}`:
   - Buka DevTools → Network → klik request ke `/api/v1/work-orders`
   - Lihat tab Headers → Request Headers

2. Cek isi `sessionStorage['auth_token']` di konsol browser:
   ```js
   sessionStorage.getItem('auth_token')
   ```

3. Cek middleware `VerifyRosteringToken` di backend maintenance.

4. Cek apakah backend rostering bisa diakses dari backend maintenance:
   ```bash
   curl http://localhost:8001/api/auth/me -H "Authorization: Bearer TOKEN"
   ```

5. Cek `DEV_MOCK_AUTH` di `.env` backend maintenance (harus `false` untuk mode SSO).

---

### Masalah: Database connection error

**Solusi:**

1. Pastikan PostgreSQL sedang berjalan.
2. Cek konfigurasi di `.env`:
   ```dotenv
   DB_HOST=localhost
   DB_PORT=5432
   DB_DATABASE=atoms_maintenance
   DB_USERNAME=postgres
   DB_PASSWORD=password_anda
   ```
3. Pastikan database sudah dibuat:
   ```sql
   CREATE DATABASE atoms_maintenance;
   ```
4. Test koneksi:
   ```bash
   rtk php artisan migrate:status
   ```

---

### Masalah: Data roster tidak muncul di dashboard maintenance

**Kemungkinan penyebab:**

| Penyebab | Solusi |
|---|---|
| Koneksi read-only ke `atoms_rostering` tidak bisa terhubung | Cek `ROSTERING_DB_*` di `.env` backend maintenance |
| Filter tanggal + shift tidak sesuai | Data roster harus dipublish di atoms-rostering untuk tanggal dan shift yang sedang aktif |
| Roster belum dipublish di atoms-rostering | Login ke atoms-rostering dan publish roster untuk bulan/tanggal yang relevan |
| Backend maintenance berjalan di timezone UTC | Sudah diatasi — frontend mengirim `date` dan `shift_type` secara eksplisit ke backend |

---

## 15. Catatan Deployment Server Kantor

Saat deploy ke server kantor, perhatikan hal-hal berikut:

### Rekomendasi domain/subdomain

| Komponen | Lokal | Contoh Production |
|---|---|---|
| atoms-rostering Frontend | `http://localhost:5174` | `https://rostering.airnav.id` |
| atoms-rostering Backend | `http://localhost:8001` | `https://api-rostering.airnav.id` |
| atoms-maintenance Frontend | `http://localhost:5173` | `https://maintenance.airnav.id` |
| atoms-maintenance Backend | `http://localhost:8000` | `https://api-maintenance.airnav.id` |

> URL production di atas adalah **contoh**. Sesuaikan dengan konfigurasi server dan DNS kantor.

### Konfigurasi env production

**Backend atoms-maintenance (`.env`):**
```dotenv
APP_ENV=production
APP_DEBUG=false
APP_URL=https://api-maintenance.airnav.id

DB_DATABASE=atoms_maintenance
# (konfigurasi DB production sesuai server)

ROSTERING_API_URL=https://api-rostering.airnav.id
ROSTERING_FRONTEND_URL=https://rostering.airnav.id

DEV_MOCK_AUTH=false

SANCTUM_STATEFUL_DOMAINS=maintenance.airnav.id
SESSION_DOMAIN=.airnav.id
```

**Frontend atoms-maintenance (`.env`):**
```dotenv
VITE_API_URL=https://api-maintenance.airnav.id/api
VITE_DEV_MOCK_AUTH=false
VITE_ROSTERING_FRONTEND_URL=https://rostering.airnav.id
```

**MenuGrid.tsx di atoms-rostering frontend:**

Ganti URL hardcoded `http://localhost:5173` dengan URL production maintenance frontend.

**Backend atoms-maintenance `config/cors.php`:**
```php
'allowed_origins' => ['https://maintenance.airnav.id'],
```

### Aturan keamanan production

- ✅ Gunakan **HTTPS** untuk semua URL
- ✅ Nonaktifkan `APP_DEBUG=false` di production
- ✅ Pastikan `db_rostering` tetap **read-only** dari sisi maintenance
- ✅ Token tetap disimpan di **sessionStorage** (bukan localStorage)
- ✅ Jangan expose token di log server
- ✅ Jangan buat JWT — gunakan Sanctum opaque token
- ✅ Pastikan CORS hanya mengizinkan origin yang benar
- ❌ Jangan simpan kredensial database di kode — gunakan `.env`

### Build frontend untuk production

```bash
# Build frontend atoms-maintenance
cd atoms-maintenance/frontend_atoms-maintenance
rtk npm run build
# Hasil build ada di folder dist/ — upload ke web server atau CDN

# Build frontend atoms-rostering (jika diperlukan)
cd atoms-rostering/frontend_atoms
rtk npm run build
```

---

## 16. Urutan Singkat Menjalankan Semua Service Lokal

Buka **4 terminal** secara bersamaan:

**Terminal 1 — Backend atoms-rostering:**
```bash
cd atoms-rostering/backend_atoms
rtk php artisan serve --host=127.0.0.1 --port=8001
```

**Terminal 2 — Frontend atoms-rostering:**
```bash
cd atoms-rostering/frontend_atoms
rtk npm run dev -- --host 127.0.0.1 --port 5174
```

**Terminal 3 — Backend atoms-maintenance:**
```bash
cd atoms-maintenance/backend_atoms-maintenance
rtk php artisan serve --host=127.0.0.1 --port=8000
```

**Terminal 4 — Frontend atoms-maintenance:**
```bash
cd atoms-maintenance/frontend_atoms-maintenance
rtk npm run dev -- --host 127.0.0.1 --port 5173
```

Setelah semua service jalan:
1. Buka `http://localhost:5174` di browser
2. Login dengan akun yang valid
3. Klik card "Maintenance"
4. Masuk ke `http://localhost:5173` tanpa login ulang

---

## 17. Checklist Akhir

Gunakan checklist ini setiap kali setup ulang atau deploy:

```
SETUP
[ ] Database atoms_maintenance sudah dibuat di PostgreSQL
[ ] Database atoms_rostering sudah dibuat dan terisi data (dikelola oleh atoms-rostering)
[ ] .env backend maintenance sudah dikonfigurasi (DB, ROSTERING_API_URL, DEV_MOCK_AUTH)
[ ] .env frontend maintenance sudah dikonfigurasi (VITE_API_URL, VITE_ROSTERING_FRONTEND_URL)
[ ] php artisan migrate sudah dijalankan di backend maintenance
[ ] php artisan key:generate sudah dijalankan
[ ] php artisan local-users:sync sudah dijalankan (opsional, untuk sinkronisasi user)

SERVICE
[ ] Backend atoms-rostering aktif di port 8001
[ ] Frontend atoms-rostering aktif di port 5174
[ ] Backend atoms-maintenance aktif di port 8000
[ ] Frontend atoms-maintenance aktif di port 5173

SSO / INTEGRASI
[ ] Menu Maintenance di atoms-rostering mengirim token lewat URL (?token=...)
[ ] Token tersimpan di sessionStorage atoms-maintenance (bukan localStorage)
[ ] Setiap request dari maintenance ke backend menyertakan Authorization: Bearer
[ ] Backend maintenance bisa memanggil /api/auth/me di rostering (port 8001)
[ ] /api/auth/me mengembalikan data user yang valid

FUNGSIONALITAS
[ ] Dashboard maintenance bisa dibuka dan menampilkan data roster aktif
[ ] Work Order page bisa dibuka
[ ] Work Order bisa dibuat (form create berfungsi)
[ ] Work Order bisa diedit
[ ] Signature panel berfungsi dan immutable setelah ditandatangani
[ ] Logout mengarahkan kembali ke halaman login atoms-rostering
```

---

## Referensi Dokumen Teknis Lainnya

| Dokumen | Lokasi | Isi |
|---|---|---|
| Master agent guide | `.agents/agents.md` | Arsitektur lengkap, rules, integration status |
| SSO rules | `.agents/instructions/sso-rules.md` | Aturan teknis SSO, token lifecycle |
| Work Order rules | `.agents/instructions/workorder-rules.md` | Aturan bisnis Work Order, signature, format nomor |
| Backend context | `atoms-maintenance/backend_atoms-maintenance/BACKEND_CONTEXT.md` | Detail API, filter, database, command |
| Rostering integration | `atoms-maintenance/backend_atoms-maintenance/docs/ROSTERING_INTEGRATION.md` | Panduan integrasi DB dan API rostering |
| Session handoff | `.agents/session-handoff.md` | Status implementasi terkini |

---

*Dokumen ini dibuat pada 2026-05-17. Sesuaikan dengan perkembangan project.*
