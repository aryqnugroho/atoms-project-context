# Project Boundary — ATOMS PROJECT

> Mendefinisikan batas zona kerja dan aturan akses untuk AI coding assistant.

---

## Zone Map

| Zone | Path | Access Level |
|------|------|-------------|
| Active Development | `atoms-maintenance/` | ✅ Full read/write |
| Frontend | `atoms-maintenance/frontend_atoms-maintenance/` | ✅ Full read/write |
| Backend | `atoms-maintenance/backend_atoms-maintenance/` | ✅ Full read/write |
| Rostering (SSO Reference) | `atoms-rostering/` | 🔴 READ-ONLY — No writes, ever |
| Rostering Exception | `atoms-rostering/frontend_atoms/src/components/feature/home/MenuGrid.tsx` | ⚠️ Satu-satunya file yang boleh dimodifikasi di rostering — hanya untuk tombol Maintenance SSO redirect |

---

## The `atoms-rostering` Rule

`atoms-rostering` adalah **primary SSO application** dan source of truth untuk:
- User login dan authentication
- Employee master records
- Shift dan roster schedules
- Leave dan shift swap requests

**Jangan pernah:**
- Write file apapun di dalam `atoms-rostering/` (kecuali exception di atas)
- Edit file apapun di dalam `atoms-rostering/` (kecuali exception di atas)
- Stage atau commit perubahan dari `atoms-rostering/` (kecuali exception di atas)
- Jalankan destructive commands di dalam `atoms-rostering/`

**Boleh:**
- Read files di dalam `atoms-rostering/` untuk referensi
- Referensikan pattern dari `backend_atoms` dalam implementasi
- Deskripsikan apa yang `atoms-rostering` expose (APIs, models, DB tables) dalam dokumentasi

---

## Tech Stack Per Zone

| Zone | Stack |
|------|-------|
| `frontend_atoms-maintenance` | React 19 + TypeScript + Vite 8 + Tailwind CSS 3.4 |
| `backend_atoms-maintenance` | PHP 8.x (Laravel) + PostgreSQL (`atoms_maintenance`) |
| `atoms-rostering` | Laravel + React + PostgreSQL (`atoms_rostering`) — read-only reference |

---

## Database Boundaries

| Database | Access dari atoms-maintenance | Aturan |
|----------|-------------------------------|--------|
| `atoms_maintenance` (db_maintenance) | ✅ Read + Write | Owned by atoms-maintenance |
| `atoms_rostering` (db_rostering) | 🔴 Read-only via koneksi `rostering` | Jangan pernah write. Gunakan `DB::connection('rostering')` |

**Jangan pernah:**
- Query `personal_access_tokens` langsung dari `atoms_rostering`
- Write ke tabel apapun di `atoms_rostering`
- Buat migration yang menyentuh `atoms_rostering`

---

## Auth Boundaries

| Aspek | Owner | Aturan |
|-------|-------|--------|
| Login / password | atoms-rostering | atoms-maintenance tidak punya login sendiri |
| Token issuance | atoms-rostering (Sanctum) | atoms-maintenance hanya memvalidasi token |
| Token validation | atoms-maintenance via proxy | Panggil `GET http://localhost:8001/api/auth/me` |
| Token storage | sessionStorage (frontend) | Jangan localStorage |
| Token format | Sanctum opaque `{id}\|{random}` | Jangan JWT |

---

## What `atoms-maintenance` Does NOT Own

- Login / authentication system (owned by `atoms-rostering`)
- Employee master data (owned by `atoms-rostering`)
- Shift dan roster schedules (owned by `atoms-rostering`)
- Roster building / management UI
- Leave request forms
- Shift swap workflows

`atoms-maintenance` **mengkonsumsi** data ini via SSO atau read-only DB connection. Tidak mereplikasi atau meng-override.

---

## Ground Check Rule

Ground Check adalah **digital logbook** — form entry langsung, bukan file upload.

**Jangan pernah:**
- Buat upload logic untuk Ground Check
- Buat file storage untuk Ground Check records
- Buat PDF upload endpoint untuk Ground Check

---

## Signature System Rule

Semua signature di semua modul menggunakan **base64 PNG di database**, bukan file upload.

**Jangan pernah:**
- Simpan signature sebagai file di filesystem
- Buat upload endpoint untuk signature
- Overwrite signature yang sudah tersimpan (immutable)
