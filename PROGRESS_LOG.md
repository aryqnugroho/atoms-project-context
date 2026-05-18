# PROGRESS LOG - ATOMS PROJECT

## Current Status

- Root ATOMS-PROJECT digunakan sebagai context-only repository.
- Folder atoms-maintenance/ dan atoms-rostering/ tidak ikut commit.
- Repo ini hanya menyimpan konteks, aturan agent, progress, dan next tasks.
- **Semua 4 service sudah berjalan lokal.**
- **SSO redirect Maintenance sudah diperbaiki dan berfungsi.**
- **TFP-005 Tower module sudah live (2026-05-18).**
- **TFP-007 Localizer module sudah live (2026-05-18).**

## Progress Entries

### feat(tfp): TFP-005 Performance Check Gedung Tower (2026-05-18)

**Module baru:** Performance Check Gedung Tower — TFP-005

**Backend:**
- 4 migrations: `tfp_tower_records`, `_technicians`, `_items`, `_facilities`
- 4 models, Template (23 parameter, 14 fasilitas), Service, Controller, 3 Requests, Exception
- 8 routes di `/api/v1/tfp/tower`
- Form number: `TFP-TOWER-YYMMDD-SEQ`

**Panel kolom (11):** Panel A 10, A 11, Panel ATS (A 13) Input/Output, A 14, A 16, A 17, A 18, A 19, A 20, Panel MILAT RU 12 & 13

**23 Parameter:** L1-N s/d Frekuensi (rows 1-12), Power Factor, Battery (14-17), Mode/Suplai (18-19), KWH Meter (20), Suhu Tower Lt 11, Suhu Ruang RX, Suhu Cabin Tower (21-23)

**14 Fasilitas:** Catu Daya Listrik, Penerangan, Rotating Beacon, Hazard Beacon, AC 22/23 (Split Wall), Pompa Air Lt 5 Tower, Lift, APAR, Atap, Plafond, Dinding, Pintu, Door Lock

**Frontend:** Types, Service, ListPage, DetailPage, PrintView, SignaturePanel, mockData TFP-005 aktif, TFP_ACTIVE_ROUTES, router

**Test:** migrate ✅ 4 tables | route:list ✅ 8 routes | test ✅ 2 passed | build ✅

**Commit:** Backend `aba91d1` ✅ | Frontend `0d99c2b` ✅ | Pushed ✅

## Progress Entries

### feat(tfp): TFP-003 Performance Check Gedung (Transmitter) TX (2026-05-18)

**Module baru:** Performance Check Gedung (Transmitter) TX — TFP-003

**Backend (atoms-maintenance/backend_atoms-maintenance):**
- 4 migrations: `tfp_transmitter_tx_records`, `_technicians`, `_items`, `_facilities`
- 4 models: `TfpTransmitterTxRecord`, `TfpTransmitterTxTechnician`, `TfpTransmitterTxItem`, `TfpTransmitterTxFacility`
- Template: `TfpTransmitterTxTemplate` — 21 parameter, 19 fasilitas
- Service: `TfpTransmitterTxService` — create/update/sign/delete + roster integration
- Controller: `TfpTransmitterTxController` — 8 endpoints
- Requests: Create/Update/Sign
- Exception: `TfpTransmitterTxDuplicateException`
- Routes: 8 routes di `/api/v1/tfp/transmitter-tx`
- Form number: `TFP-TX-YYMMDD-SEQ` (contoh: `TFP-TX-260519-001`)

**Frontend (atoms-maintenance/frontend_atoms-maintenance):**
- Types: `src/types/tfpTransmitterTx.ts`
- Service: `src/services/tfpTransmitterTxService.ts`
- Pages: `TfpTransmitterTxListPage`, `TfpTransmitterTxDetailPage`, `TfpTransmitterTxPrintView`
- Component: `TfpTransmitterTxSignaturePanel`
- mockData: TFP-003 `is_active_mvp: true`
- TfpIndexPage: `TFP-003` ditambahkan ke `TFP_ACTIVE_ROUTES`
- Router: list/detail/print routes untuk Transmitter TX

**Panel kolom (11 kolom):**
Panel TX 01, Panel TX 02, Panel COS (TX 03) Input/Output, Panel Output UPS TX 04, Panel UPS (TX 07) Input/Output, Panel AC TX 06, UPS PILLER Input/Output, Panel MILAT RU 11

**Fasilitas (19):** Catu Daya Listrik, Penerangan, Obstacle Light, UPS 30 KVA PILLER, ETS 30 KVA (U/S), AC 02-07 (Split Wall), Exhaust Fan, Papan Nama AirNav, Rumput, APAR/Fire Extinguisher, Atap, Dinding, Pintu (×2)

**Test:** migrate ✅ 4 tables | route:list ✅ 24 TFP routes | test ✅ 2 passed | build ✅

**Commit:**
- Backend: `a685aa6` — pushed ✅
- Frontend: `f939eb1` — pushed ✅

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
