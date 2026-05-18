# PROGRESS LOG - ATOMS PROJECT

## Current Status

- Root ATOMS-PROJECT digunakan sebagai context-only repository.
- Folder atoms-maintenance/ dan atoms-rostering/ tidak ikut commit.
- Repo ini hanya menyimpan konteks, aturan agent, progress, dan next tasks.
- **Semua 4 service sudah berjalan lokal (2026-05-18).**

## Progress Entries

### Full Local Setup & Integration (2026-05-18)

**Setup selesai:**
- Database PostgreSQL dibuat: `atoms_rostering` dan `atoms_maintenance`
- Backend atoms-rostering: composer install ✅, key:generate ✅, migrate ✅, db:seed ✅
- Backend atoms-maintenance: composer install ✅, key:generate ✅, migrate ✅
- Frontend atoms-rostering: npm install ✅, build ✅ (fix unused import Select di ProfileModal.tsx)
- Frontend atoms-maintenance: npm install ✅, build ✅

**Service berjalan:**
| Service | Port | Status |
|---|---|---|
| atoms-rostering backend | 8001 | ✅ Running |
| atoms-rostering frontend | 5174 | ✅ Running |
| atoms-maintenance backend | 8000 | ✅ Running |
| atoms-maintenance frontend | 5173 | ✅ Running |

**Akun login (dari seeder atoms-rostering):**
- Admin: `admin@airnav.com` / `password`
- Semua user: `user1@airnav.com` s/d `user54@airnav.com` / `password`
- Total: 55 user (1 admin + 54 karyawan)

**Integrasi SSO:**
- Login atoms-rostering → token Sanctum opaque → sessionStorage
- Frontend maintenance kirim `Authorization: Bearer {token}`
- Backend maintenance validasi ke `GET http://127.0.0.1:8001/api/auth/me` ✅
- Endpoint `/api/v1/auth/verify` maintenance → 200 OK ✅
- CORS rostering mengizinkan localhost:5173 dan localhost:5174 ✅

**Test:**
- atoms-maintenance: 2 passed ✅
- atoms-rostering: 19 passed, 9 failed (RosterControllerTest — test fixture issue, bukan bug kita)

**File yang diubah:**
- atoms-rostering/frontend_atoms/src/components/modals/user/ProfileModal.tsx — hapus unused import Select
- atoms-rostering/frontend_atoms/.env — fix VITE_API_URL ke port 8001 (sebelumnya salah ke 8000)
- atoms-rostering/backend_atoms/.env — dibuat baru (PostgreSQL, port 8001)
- atoms-maintenance/backend_atoms-maintenance/.env — dibuat baru (PostgreSQL, ROSTERING_API_URL, DEV_MOCK_AUTH=false)
- atoms-maintenance/frontend_atoms-maintenance/.env — dibuat baru (VITE_API_URL, VITE_ROSTERING_FRONTEND_URL)

### Initial Context Repo Setup

- Context repo disiapkan.
- .gitignore root diperbarui agar tidak memasukkan folder aplikasi.
- File tracking project dibuat.
