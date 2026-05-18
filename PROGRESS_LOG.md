# PROGRESS LOG - ATOMS PROJECT

## Current Status

- Root ATOMS-PROJECT digunakan sebagai context-only repository.
- Folder atoms-maintenance/ dan atoms-rostering/ tidak ikut commit.
- Repo ini hanya menyimpan konteks, aturan agent, progress, dan next tasks.
- **Semua 4 service sudah berjalan lokal.**
- **SSO redirect Maintenance sudah diperbaiki dan berfungsi.**

## Progress Entries

### Fix: Maintenance Menu SSO Redirect (2026-05-18)

**Masalah:** Card Maintenance di atoms-rostering menggunakan `navigate('/maintenance')` yang mengarah ke halaman internal "Coming Soon" di atoms-rostering, bukan ke atoms-maintenance.

**Root cause:** `MenuGrid.tsx` mendefinisikan `route: '/maintenance'` dan menggunakan `useNavigate()` untuk semua menu item, termasuk Maintenance. Tidak ada SSO token handoff.

**Perbaikan:**
- `MenuGrid.tsx` — tambahkan `handleItemClick()` yang mendeteksi item Maintenance secara khusus
- Saat Maintenance diklik: baca token dari `sessionStorage['auth_token']` via `getStoredToken()`
- Redirect ke `VITE_MAINTENANCE_URL?token={encodeURIComponent(token)}`
- Jika tidak ada token: redirect ke `VITE_MAINTENANCE_URL` saja (atoms-maintenance akan redirect ke rostering login)
- Tambahkan `VITE_MAINTENANCE_URL=http://localhost:5173` ke `.env` dan `.env.example`

**File yang diubah:**
- `atoms-rostering/frontend_atoms/src/components/feature/home/MenuGrid.tsx`
- `atoms-rostering/frontend_atoms/.env` (tambah VITE_MAINTENANCE_URL)
- `atoms-rostering/frontend_atoms/.env.example` (dibuat baru)

**Commit:** `94a5730` di atoms-rostering/frontend_atoms

**Smoke test:**
- Login rostering: ✅ 200 OK
- Rostering `/api/auth/me`: ✅ 200 OK
- Maintenance `/api/v1/auth/verify`: ✅ 200 OK, user: Administrator/Admin
- Build rostering frontend: ✅
- Build maintenance frontend: ✅

---

### Full Local Setup & Integration (2026-05-18)

**Setup selesai:**
- Database PostgreSQL dibuat: `atoms_rostering` dan `atoms_maintenance`
- Backend atoms-rostering: composer install ✅, key:generate ✅, migrate ✅, db:seed ✅
- Backend atoms-maintenance: composer install ✅, key:generate ✅, migrate ✅
- Frontend atoms-rostering: npm install ✅, build ✅
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

**Test:**
- atoms-maintenance: 2 passed ✅
- atoms-rostering: 19 passed, 9 failed (RosterControllerTest — test fixture issue, bukan bug kita)

---

### Initial Context Repo Setup

- Context repo disiapkan.
- .gitignore root diperbarui agar tidak memasukkan folder aplikasi.
- File tracking project dibuat.
