---
inclusion: auto
name: atoms-project-root
description: Root-level steering for the ATOMS PROJECT workspace. Auto-loaded for all tasks.
---

# ATOMS PROJECT — Root Steering

> Root-level Kiro steering file. Auto-loaded untuk semua tasks.
> Panduan agent yang otoritatif ada di `.agents/agents.md`.

## → Baca Ini Dulu

**[`KIRO_TASK_CONTEXT.md`](../../KIRO_TASK_CONTEXT.md)** — **Wajib dibaca sebelum task pengembangan apapun.** Single source of truth ringkas: hard constraints, SSO, signature, Work Order, CNSD, dan pola TFP.

**[`.kiro/steering/rtk-rules.md`](./rtk-rules.md)** — **Wajib dibaca sebelum menjalankan terminal command apapun.** RTK menghemat 60–90% token.

**[`.kiro/hooks/manual-form-workflow.md`](../hooks/manual-form-workflow.md)** — **Wajib dibaca sebelum membuat form manual baru.** Checklist lengkap: pull, backend, frontend, roster, signature, print view, testing, docs, commit, push.

**[`.agents/agents.md`](../../.agents/agents.md)** — Master agent guide: arsitektur, rules, integration status, hard constraints.

**[`.agents/session-handoff.md`](../../.agents/session-handoff.md)** — Status task terakhir dan next steps.

## Workspace Zones

| Zone | Status |
|------|--------|
| `atoms-maintenance/` | ✅ Active development |
| `atoms-rostering/` | 🔴 READ-ONLY — no writes ever (kecuali `MenuGrid.tsx` untuk SSO button) |

## Integration Status (per 2026-05-19 — CNSD Receiver Meter Reading live)

| Integration | Status |
|-------------|--------|
| Work Orders (CRUD, sign, print) | ✅ Complete & Verified |
| Work Order list filter (search, tanggal, tahun, divisi, shift, status) | ✅ Implemented |
| Work Order list filter UI (chips, reset, result count, empty state) | ✅ Implemented |
| Work Order nomor format baru `WO-DIV-YYYYMMDD-SEQ` | ✅ Implemented |
| Database default kosong (no dummy seed) | ✅ Implemented |
| Bulk sync local_users (`php artisan local-users:sync`) | ✅ Implemented |
| Cleanup stale local_users (`php artisan local-users:cleanup`) | ✅ Implemented |
| Rostering DB read-only connection | ✅ Implemented |
| Shift personnel API (filter date + shift) | ✅ Implemented |
| SSO (token proxy via rostering API) | ✅ Implemented |
| Dashboard: Manager Teknik + Supervisor CNSD/TFP + Teknisi per divisi | ✅ Implemented |
| Work Order create UI submit (pre-validation + auto-fill personnel) | ✅ Implemented |
| Work Order edit UI submit + immutable field lock | ✅ Implemented |
| Work Order detail page refresh on focus + after mutation | ✅ Implemented |
| Signature authorization name-match (403 jika nama tidak cocok) | ✅ Implemented |
| Print layout (No+Nama, checklist `✓`) | ✅ Updated |
| **CNSD Readiness Form EQ-1 (Phase 4 pilot)** | ✅ **Live** |
| - Backend: migrations, models, service, request, controller, routes | ✅ Implemented |
| - EQ-1 item template (5 sections, 36 items) | ✅ Implemented |
| - Auto-generate items + auto-attach personnel from rostering | ✅ Implemented |
| - Per-technician signature row (immutable, name-matched, no delegation) | ✅ Implemented |
| - Frontend: list page, detail/edit page, signature panel, dummy form removed | ✅ Implemented |
| - **Print view (`/cnsd/readiness/:id/print`)** | ✅ Implemented |
| - **Tombol print di list + detail, badge "Perlu TTD" di list** | ✅ Implemented |
| - **Notification on create + completed (CnsdReadinessCreated/Completed)** | ✅ Implemented |
| **CNSD Radar Meter Reading (Phase 4 Module 2)** | ✅ **Live (2026-05-18)** |
| - Backend: 3 migrations, 3 models, service, template (48 items), controller, requests | ✅ Implemented |
| - Form-number `RADAR-YYMMDD-SEQ` | ✅ Implemented |
| - Frontend: list page, detail/edit (TX dual + environment layouts), signature panel, print view | ✅ Implemented |
| - Card Radar di CNSD index sekarang aktif (CNSD-002) | ✅ Implemented |
| **CNSD Recorder Meter Reading (Phase 4 Module 3)** | ✅ **Live (2026-05-19)** |
| - Backend: 3 migrations, 3 models, service, template (72 items, 15 U/S blocked), controller, requests | ✅ Implemented |
| - Form-number `RECORDER-YYMMDD-SEQ` (FORM C-3) | ✅ Implemented |
| - Frontend: list page, detail/edit (Server A/B + environment layouts), signature panel, print view | ✅ Implemented |
| - Card Recorder di CNSD index sekarang aktif (CNSD-003) | ✅ Implemented |
| - U/S item rendering (red strip, disabled inputs, backend update guard) | ✅ Implemented |
| Other CNSD cards (Receiver, Glide Path, Localizer, T-DME, DVOR, DME, ATC System, ATIS) | 🚧 Coming Soon |
| **Grounding Report (2026-05-19)** | ✅ **Live** |
| - Backend: 3 migrations, 3 models, service, template (6 VISUAL + 3 PENGUKURAN), controller, requests | ✅ Implemented |
| - Report-number `GROUNDING-YYMMDD-SEQ` | ✅ Implemented |
| - Roster: TFP (Support), Supervisor TFP, Manager Teknik | ✅ Implemented |
| - Frontend: list page (API-connected, create modal), detail/edit, signature panel, print view | ✅ Implemented |
| - Hapus dropdown Kantor Unit Kerja, hapus Dibuat Oleh/Disetujui Oleh | ✅ Implemented |
| - Print view: AirNav logo, TFP-style footer signature, manual only | ✅ Implemented |
| **CNSD Transmitter Meter Reading (Phase 4 Module 5)** | ✅ **Live (2026-05-19)** |
| - Backend: 3 migrations, 3 models, service, template (9 TX groups + 4 env items), controller, requests | ✅ Implemented |
| - Form-number `TRANSMITTER-YYMMDD-SEQ` (FORM C-1) | ✅ Implemented |
| - Frontend: list page, detail/edit (Transmitter + Lingkungan Kerja tabs), signature panel, print view | ✅ Implemented |
| - Card Transmitter di CNSD index sekarang aktif (CNSD-005) | ✅ Implemented |
| - Back Up Radio status disabled/blocked (grey cells, backend rejects update) | ✅ Implemented |
| **CNSD Receiver Meter Reading (Phase 4 Module 6)** | ✅ **Live (2026-05-19)** |
| - Backend: 3 migrations, 3 models, service, template (9 receiver groups + 4 env items), controller, requests | ✅ Implemented |
| - Form-number `RECEIVER-YYMMDD-SEQ` (FORM C-2) | ✅ Implemented |
| - Frontend: list page, detail/edit (Receiver + Lingkungan Kerja tabs), signature panel, print view | ✅ Implemented |
| - Card Receiver di CNSD index sekarang aktif (CNSD-006) | ✅ Implemented |
| - Status A/B dropdown: ON LINE / OFF LINE, Sequelsh On free text | ✅ Implemented |
| Other CNSD cards (Glide Path, Localizer, T-DME, DVOR, DME, ATC System, ATIS) | 🚧 Coming Soon |
| **TFP Performance Check AOB Lantai Ground (Phase 5 Module 1)** | ✅ **Live (2026-05-19)** |
| - Backend: 4 migrations, 4 models, service, template (21 params + 17 facilities), controller, requests | ✅ Implemented |
| - Form-number `TFP-AOBLTGND-YYMMDD-SEQ` | ✅ Implemented |
| - Roster: employee_type='Support', Supervisor TFP, Manager Teknik | ✅ Implemented |
| - Frontend: list page, detail/edit (disabled cells, dropdowns), signature panel, print view | ✅ Implemented |
| - Card AOB Ground di TFP index sekarang aktif (TFP-001) | ✅ Implemented |
| - Row "Suhu Ruang ARO" DIHAPUS per permintaan | ✅ Implemented |
| Other TFP cards (AOB Lantai 1 & 2, Transmitter, Radar, Tower, VOR, Localizer, Glide Path) | 🚧 Coming Soon |
| **TFP Performance Check AOB Lantai 1 & 2 (Phase 5 Module 2)** | ✅ **Live (2026-05-19)** |
| - Backend: 4 migrations, 4 models, service, template (21 params + 24 facilities), controller, requests | ✅ Implemented |
| - Form-number `TFP-AOBLT12-YYMMDD-SEQ` | ✅ Implemented |
| - 6 single-value panel columns (no Input/Output split): A05, A06, A07, A08, A22, A09 | ✅ Implemented |
| - Frontend: list page, detail/edit, signature panel, print view | ✅ Implemented |
| - Card AOB Lantai 1 & 2 di TFP index sekarang aktif (TFP-002) | ✅ Implemented |
| Other TFP cards (Transmitter, Radar, Tower, VOR, Localizer, Glide Path) | 🚧 Coming Soon |

## Hard Constraints (Jangan Dilanggar)

- Jangan gunakan JWT — Sanctum opaque token
- Jangan simpan token di localStorage — gunakan sessionStorage
- Jangan query `personal_access_tokens` langsung — proxy via `/api/auth/me`
- Jangan write ke `db_rostering`
- Jangan buat upload logic untuk Ground Check (digital logbook)
- Jangan overwrite signature yang sudah tersimpan (immutable)
- Jangan ubah status Work Order: `ongoing`, `on_hold`, `completed` adalah final

## Module-Level Steering

- **Frontend UI design:** [`frontend_atoms-maintenance/.kiro/steering/ui-design.md`](../../atoms-maintenance/frontend_atoms-maintenance/.kiro/steering/ui-design.md)

## Skills Available

| Skill | Canonical Location | Scope |
|-------|--------------------|-------|
| Backend architecture | `backend_atoms-maintenance/.kiro/skills/backend-architecture/SKILL.md` | Backend tasks |
| Frontend architecture | `frontend_atoms-maintenance/.kiro/skills/frontend-architecture/SKILL.md` | Frontend tasks |
| UI design | `.agents/instructions/ui-design.md` | UI polish tasks |
| Git auto-ship | `atoms-maintenance/.kiro/skills/git-auto-ship/SKILL.md` | Commit & push |
| Impeccable | `frontend_atoms-maintenance/.kiro/skills/impeccable/SKILL.md` | UI audit |

## Key Context Files

| Purpose | Location |
|---------|----------|
| **Panduan menjalankan & deploy** | **`CARA_MENJALANKAN_ATOMS_MAINTENANCE.md`** (root project) |
| Getting started guide | `.agents/GETTING_STARTED.md` |
| Session handoff | `.agents/session-handoff.md` |
| Project boundaries | `.agents/project-boundary.md` |
| **SSO rules** | `.agents/instructions/sso-rules.md` |
| Signature system rules | `.agents/instructions/signature-system-rules.md` |
| Work order rules | `.agents/instructions/workorder-rules.md` |
| UI design instructions | `.agents/instructions/ui-design.md` |
| Rostering integration guide | `atoms-maintenance/backend_atoms-maintenance/docs/ROSTERING_INTEGRATION.md` |

## Form Development Hooks

Untuk task pembuatan atau perbaikan form manual, baca hook yang sesuai:

| Task | Hook Wajib | Instruction Tambahan |
|------|-----------|----------------------|
| **Form manual baru (semua domain)** | **[`.kiro/hooks/manual-form-workflow.md`](../hooks/manual-form-workflow.md)** | Checklist lengkap dari pull hingga push |
| Task CNSD | `.kiro/hooks/manual-form-workflow.md` + `.agents/instructions/cnsd-readiness-rules.md` | Pola EQ-1 sebagai referensi utama |
| Task TFP | `.kiro/hooks/manual-form-workflow.md` + `.agents/instructions/tfp-rules.md` | Pola AOB Ground sebagai referensi |
| Task Ground Check / Grounding | `.kiro/hooks/manual-form-workflow.md` | Baca modul CNSD/TFP yang paling mirip |

**Aturan wajib untuk semua task form:**
- Selalu pull backend/frontend/context sebelum coding
- Selalu update docs setelah task selesai
- Selalu commit dan push context setelah task selesai
- Dynamic form tetap **HOLD** — jangan diimplementasikan

## Terminal Commands

> ⚠️ **Baca `.kiro/steering/rtk-rules.md` sebelum menjalankan terminal command apapun.**

Gunakan prefix `rtk` untuk semua terminal commands yang menghasilkan output besar.
RTK menghemat 60–90% token pada git, npm, composer, php artisan, docker, dll.

```bash
# Contoh wajib RTK:
rtk git status              # bukan: git status
rtk git diff                # bukan: git diff
rtk test php artisan test   # bukan: php artisan test
rtk npm run build           # bukan: npm run build
rtk php artisan route:list  # bukan: php artisan route:list
rtk composer install        # bukan: composer install
```

Jangan gunakan RTK untuk built-in tools Kiro (read_file, grep_search, list_directory) — sudah dioptimasi.
Gunakan `control_pwsh_process` untuk long-running processes (serve, dev, queue:work).
