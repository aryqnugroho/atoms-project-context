# AGENTS.md — ATOMS PROJECT (Root)

> Master entry point untuk AI coding assistant yang bekerja di ATOMS PROJECT workspace.
> Baca file ini sebelum memulai task apapun.

---

## Project Overview

- **Project:** ATOMS PROJECT — Airport Technical Operations Management System
- **Client:** AirNav Indonesia, Surabaya Branch
- **Working Zone:** `atoms-maintenance/` — zona kerja aktif
- **Reference Zone:** `atoms-rostering/` — READ-ONLY. Jangan pernah dimodifikasi.

---

## Two-System Architecture

```
┌──────────────────────────┐     ┌──────────────────────────────┐
│  atoms-rostering         │     │  atoms-maintenance           │
│  (Source of Truth)       │     │  (Active Development)        │
│                          │     │                              │
│  • User accounts/login   │◄────│  • Consumes auth via SSO     │
│  • Employee master data  │     │  • Work Orders ✅ Complete   │
│  • Shift assignments     │     │  • CNSD inspections (EQ-1)   │
│  • Roster schedules      │     │  • TFP perf checks (AOB)     │
│  • Leave/shift requests  │     │  • Ground Check (logbook)    │
│                          │     │  • Grounding inspections     │
│  READ-ONLY REFERENCE     │     │  • Reporting & Dashboard     │
└──────────────────────────┘     └──────────────────────────────┘
```

---

## Workspace Boundaries

Lihat → [`.agents/project-boundary.md`](./project-boundary.md)

---

## Module Entry Points

| Module | Agent Guide | Context |
|--------|-------------|---------|
| Frontend (React/Vite) | [`frontend_atoms-maintenance/AGENTS.md`](../atoms-maintenance/frontend_atoms-maintenance/AGENTS.md) | [`FRONTEND_CONTEXT.md`](../atoms-maintenance/frontend_atoms-maintenance/FRONTEND_CONTEXT.md) |
| Backend (Laravel) | [`backend_atoms-maintenance/AGENTS.md`](../atoms-maintenance/backend_atoms-maintenance/AGENTS.md) | [`BACKEND_CONTEXT.md`](../atoms-maintenance/backend_atoms-maintenance/BACKEND_CONTEXT.md) |

---

## Skills Available

| Skill | Location | Applies To |
|-------|----------|------------|
| Backend Architecture | [`backend_atoms-maintenance/.kiro/skills/backend-architecture/SKILL.md`](../atoms-maintenance/backend_atoms-maintenance/.kiro/skills/backend-architecture/SKILL.md) | Laravel backend tasks |
| Frontend Architecture | [`frontend_atoms-maintenance/.kiro/skills/frontend-architecture/SKILL.md`](../atoms-maintenance/frontend_atoms-maintenance/.kiro/skills/frontend-architecture/SKILL.md) | React frontend tasks |
| UI Design | [`.agents/instructions/ui-design.md`](./instructions/ui-design.md) | UI polish dan konsistensi |
| Git Auto-Ship | [`atoms-maintenance/.kiro/skills/git-auto-ship/SKILL.md`](../atoms-maintenance/.kiro/skills/git-auto-ship/SKILL.md) | Commit & push |
| Impeccable | [`frontend_atoms-maintenance/.kiro/skills/impeccable/SKILL.md`](../atoms-maintenance/frontend_atoms-maintenance/.kiro/skills/impeccable/SKILL.md) | UI audit dan design review |

---

## Key Instructions

| Document | Location | Purpose |
|----------|----------|---------|
| **Task Context (ringkas)** | **[`KIRO_TASK_CONTEXT.md`](../KIRO_TASK_CONTEXT.md)** | **Baca ini dulu — hard constraints, SSO, signature, CNSD, TFP pattern** |
| **Manual Form Hook** | **[`.kiro/hooks/manual-form-workflow.md`](../.kiro/hooks/manual-form-workflow.md)** | **Wajib sebelum membuat form manual baru — checklist lengkap** |
| Getting Started | [`.agents/GETTING_STARTED.md`](./GETTING_STARTED.md) | Setup lokal dev |
| Project Boundary | [`.agents/project-boundary.md`](./project-boundary.md) | Batas zona kerja |
| Session Handoff | [`.agents/session-handoff.md`](./session-handoff.md) | Status task terakhir |
| SSO Rules | [`.agents/instructions/sso-rules.md`](./instructions/sso-rules.md) | Aturan SSO integration |
| Signature System Rules | [`.agents/instructions/signature-system-rules.md`](./instructions/signature-system-rules.md) | Aturan signature semua modul |
| Work Order Rules | [`.agents/instructions/workorder-rules.md`](./instructions/workorder-rules.md) | Work Order status, signing, business rules |
| CNSD Readiness Rules | [`.agents/instructions/cnsd-readiness-rules.md`](./instructions/cnsd-readiness-rules.md) | Form EQ-1, Radar, Recorder schema, template, signature, roster |
| Rostering Integration | [`backend_atoms-maintenance/docs/ROSTERING_INTEGRATION.md`](../atoms-maintenance/backend_atoms-maintenance/docs/ROSTERING_INTEGRATION.md) | Cara maintenance terhubung ke rostering |

---

## Universal Rules (berlaku untuk semua modul)

1. **Jangan pernah modifikasi `atoms-rostering/`** — zero exceptions.
2. **Baca module `AGENTS.md` sebelum setiap task.**
3. **Tidak ada secret di code** — selalu gunakan `.env` variables.
4. **Perubahan inkremental** — satu fitur per task.
5. **Validasi sebelum commit** — lint + build/test harus pass.
6. **Session handoff** — lihat [`.agents/session-handoff.md`](./session-handoff.md).
7. **Jangan gunakan JWT** — auth menggunakan Laravel Sanctum opaque token.
8. **Jangan simpan token di localStorage** — gunakan `sessionStorage`.
9. **Jangan query `personal_access_tokens` langsung** — validasi token via `GET /api/auth/me` di rostering.
10. **Jangan write ke `db_rostering`** — koneksi `rostering` di database.php adalah read-only.
11. **Database maintenance default kosong** — tidak ada dummy seed Work Order. `local_users` diisi via SSO login lazy-create, Work Order create lazy-create, atau `php artisan local-users:sync`. `MockUserSeeder` dan `WorkOrderSeeder` deprecated, tidak dipanggil dari `DatabaseSeeder`.

---

## Integration Status (per 2026-05-19)

| Integration | Status | Notes |
|-------------|--------|-------|
| SSO (rostering → maintenance) | ✅ Implemented | Token proxy via `GET /api/auth/me`. Lihat `ROSTERING_INTEGRATION.md` Section 7 |
| Rostering DB read-only connection | ✅ Implemented | Koneksi `rostering` di `config/database.php` |
| Shift personnel API (date + shift filter) | ✅ Implemented | `GET /api/v1/personnel/shift-today?date=…&shift_type=…` mengembalikan `manager`, `supervisor_cnsd`, `supervisor_tfp`, `personnel` |
| Manager Teknik di dashboard | ✅ Implemented | MT diquery dari `shift_assignments` (bukan `manager_duties` yang kosong) |
| Frontend roster filter helper | ✅ Implemented | `src/lib/shiftUtils.ts` — single source untuk shift+date dari client clock |
| Lazy local_users mapping | ✅ Implemented | `LocalUserResolver` membuat row local_users on-the-fly dari rostering saat WO create. Frontend kirim `rostering_user_id`, backend translate ke `local_users.id` |
| Bulk sync local_users | ✅ Implemented | `php artisan local-users:sync` (54 active users dari rostering). Mendukung `--dry-run` dan `--prune-stale` |
| Cleanup stale local_users | ✅ Implemented | `php artisan local-users:cleanup --dry-run|--force` aman: hanya hapus duplikat tanpa FK references |
| Work Order full lifecycle | ✅ Complete & Verified | CRUD, sign, print, status machine. Create/update verified via curl |
| Work Order create UI | ✅ Submit jalan | Frontend pre-validation + scroll-into-view error banner + dev console payload summary |
| Work Order create personnel auto-fill | ✅ Implemented | Backend auto-fill personnel array dari rostering bila frontend mengirim kosong (shift WO) |
| Work Order edit UI | ✅ Submit jalan | Edit-mode fetch WO real dari API, lock immutable fields, banner error jelas |
| WorkOrderDetailPage refresh | ✅ Implemented | Refetch on mount, on window focus, dan setelah setiap mutation (sign / feedback) |
| Work Order signature authorization | ✅ Name-matched | Backend `WorkOrderService::assertSignerCanSignRole()` membandingkan nama signer dengan nama cached di WO. Wrong name → 403 |
| Work Order print layout | ✅ Updated | Tabel personel: No+Nama saja. Checklist Output dan Pelaksanaan pakai `✓` |
| Database default kosong | ✅ Implemented | `DatabaseSeeder` empty. `MockUserSeeder` & `WorkOrderSeeder` deprecated. Setelah `migrate:fresh --seed` → 0 work_orders, 0 local_users |
| atoms-maintenance login page | ✅ Replaced | Production: redirect ke rostering. Mock form dipertahankan untuk dev |
| CNSD backend module | ✅ 3 modules live | EQ-1 (Kesiapan Peralatan) + RADAR-METER (Radar Meter Reading) + RECORDER-METER (Recorder Meter Reading, FORM C-3). Other cards remain Coming Soon. See `cnsd-readiness-rules.md` |
| TFP, Ground Check, Grounding, Logbook, Reporting | ⬜ Planned | Phase 5+ |

---

## Port Assignments (Local Dev)

| Service | Port | URL |
|---------|------|-----|
| atoms-rostering backend | 8001 | `http://localhost:8001` |
| atoms-rostering frontend | 5174 | `http://localhost:5174` (jalankan dengan `npm run dev -- --port 5174`) |
| atoms-maintenance backend | 8000 | `http://localhost:8000` |
| atoms-maintenance frontend | 5173 | `http://localhost:5173` (Vite default) |

---

## Hard Constraints (JANGAN DILANGGAR)

| Constraint | Alasan |
|-----------|--------|
| Jangan ubah logic aplikasi | Stabilitas |
| Jangan ubah database schema tanpa instruksi eksplisit | Integritas data |
| Jangan modifikasi `atoms-rostering/` kecuali tombol Maintenance di MenuGrid | Boundary |
| Jangan buat login baru di atoms-maintenance | SSO sudah implemented |
| Jangan gunakan JWT | Sanctum opaque token adalah pilihan final |
| Jangan simpan token di localStorage | Gunakan sessionStorage |
| Jangan query `personal_access_tokens` langsung | Gunakan API proxy |
| Jangan write ke `db_rostering` | Read-only |
| Jangan buat upload logic untuk Ground Check | Digital logbook, bukan file upload |
| Jangan ubah status Work Order final: `ongoing`, `on_hold`, `completed` | Business rule final |
| Jangan overwrite signature yang sudah tersimpan | Immutable by design |

## Panduan Menjalankan Project

> 📄 Lihat **[`CARA_MENJALANKAN_ATOMS_MAINTENANCE.md`](../CARA_MENJALANKAN_ATOMS_MAINTENANCE.md)** di root folder untuk panduan lengkap setup, menjalankan semua service, integrasi SSO, troubleshooting, dan deployment server kantor.
