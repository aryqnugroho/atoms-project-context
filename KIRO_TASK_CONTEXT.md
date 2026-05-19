# KIRO_TASK_CONTEXT.md — ATOMS MAINTENANCE
> Single source of truth ringkas untuk prompt Kiro. Baca ini sebelum task apapun.
> Last updated: 2026-05-19

---

## 1. Project Overview

**ATOMS MAINTENANCE** — Airport Technical Operations Management System untuk AirNav Indonesia Surabaya.

| Layer | Stack |
|---|---|
| Frontend | React 19 + TypeScript + Vite 8 + Tailwind CSS 3.4 |
| Backend | PHP 8.x (Laravel) + PostgreSQL |
| Auth | Laravel Sanctum opaque token dari atoms-rostering |
| Zona kerja | `atoms-maintenance/` — full read/write |
| Zona referensi | `atoms-rostering/` — READ-ONLY, jangan pernah dimodifikasi |

---

## 2. Core Hard Constraints

❌ Jangan ubah SSO/auth yang sudah berhasil.
❌ Jangan buat login baru di atoms-maintenance.
❌ Jangan gunakan JWT — hanya Sanctum opaque token.
❌ Jangan simpan token di localStorage — wajib sessionStorage.
❌ Jangan query `personal_access_tokens` langsung — proxy via `/api/auth/me`.
❌ Jangan write ke `db_rostering` — read-only.
❌ Jangan merusak Work Order.
❌ Jangan merusak modul CNSD yang sudah berjalan (EQ-1, Radar, Recorder).
❌ Jangan aktifkan WorkOrderSeeder default.
❌ Jangan push sebelum build/test hijau.
❌ Jangan menghapus route/controller/module hasil pull dari developer lain.
✅ Semua terminal command wajib pakai RTK (lihat `.kiro/steering/rtk-rules.md`).
✅ Jika ada controller/route missing setelah pull, investigasi dulu — jangan hapus.

---

## 3. Auth & SSO Current State

Flow SSO yang sudah berjalan:
1. User login di atoms-rostering (port 5174).
2. Klik tombol Maintenance → `MenuGrid.tsx` baca token dari sessionStorage.
3. Redirect ke `http://localhost:5173?token={sanctum_token}`.
4. `AuthContext.initAuth()` baca token dari URL, hapus dari URL.
5. Panggil `GET /api/v1/auth/verify` → backend proxy ke `GET http://localhost:8001/api/auth/me`.
6. Jika valid: simpan token di **sessionStorage** (`auth_token`), set user di context.
7. Jika invalid: redirect ke rostering login.

Dev mode: `DEV_MOCK_AUTH=true` (backend) + `VITE_DEV_MOCK_AUTH=true` (frontend) → bypass SSO.
Mock login: `http://localhost:5173/login` dengan email apapun.

---

## 4. Signature Rules

- Signature = base64 PNG data URL (`data:image/png;base64,...`).
- **Immutable** — setelah tersimpan tidak bisa diubah atau dihapus.
- **Tidak bisa diwakilkan** — signer harus user yang namanya tercatat di record.
- Backend wajib validasi: role match + name match (tolerant: trim + collapse + case-insensitive).
- Wrong name/role → 403. Already signed → 409. Invalid payload → 422.
- Trait `HasSignature` di `app/Traits/HasSignature.php` dipakai semua modul.
- Untuk CNSD: setiap teknisi punya row sendiri, sign per-row.
- Untuk Work Order: satu row per role (mt, supervisor, technician).

---

## 5. Work Order Current State

- ✅ **Stabil** — CRUD, sign, print, filter semua berjalan.
- Status: `ongoing` | `on_hold` | `completed` — di-derive dari signature, tidak diset client.
- Format nomor: `WO-{DIVISI}-{YYYYMMDD}-{SEQ}` (contoh: `WO-CNSD-20260519-001`).
- WorkOrderSeeder **deprecated** — tidak dipanggil dari DatabaseSeeder.
- Jangan merusak Work Order saat membuat modul baru.

---

## 6. CNSD Current State

**Pendekatan:** manual/template hardcoded per alat. Tidak ada dynamic form builder.

| Kode | Modul | Form | Status | Route |
|---|---|---|---|---|
| CNSD-001 | Kesiapan Peralatan | EQ-1 | ✅ Live | `/cnsd/readiness` |
| CNSD-002 | Radar Meter Reading | RADAR-METER | ✅ Live | `/cnsd/radar-meter` |
| CNSD-003 | Recorder Meter Reading | RECORDER-METER (FORM C-3) | ✅ Live | `/cnsd/recorder-meter` |
| CNSD-004 | AMSC Meter Reading | AMSC-METER | ✅ Live | `/cnsd/amsc-meter` |
| CNSD-005 | Transmitter Meter Reading | TRANSMITTER-METER (FORM C-1) | ✅ Live | `/cnsd/transmitter-meter` |
| CNSD-006 | Receiver Meter Reading | RECEIVER-METER (FORM C-2) | ✅ Live | `/cnsd/receiver-meter` |
| CNSD-007 | Glide Path Meter Reading | GLIDEPATH-METER (ILS-GP) | ✅ Live | `/cnsd/glidepath-meter` |
| CNSD-008 | Localizer Meter Reading | LOCALIZER-METER (ILS-LLZ) | ✅ Live | `/cnsd/localizer-meter` |
| CNSD-009+ | T-DME, DVOR, DME, ATC System, ATIS | — | 🚧 Coming Soon | — |

**Pola roster CNSD (berlaku untuk semua modul CNSD):**
- Ambil tanggal + shift dari request (frontend kirim dari client clock).
- `getShiftManager(shift, date)` → Manager Teknik (nullable).
- `getShiftSupervisorByDivision(shift, date, 'CNS')` → Supervisor CNSD (nullable).
- `getShiftPersonnel(shift, date)` filter `employee_type='CNS'` → semua teknisi CNSD.
- TFP/Support **tidak diikutkan**.
- Jika tidak ada teknisi CNSD → create gagal 422.
- Nama personel di-cache di record saat create (immutable).

**Format nomor form:**
- EQ-1: `EQ-1-CNSD-YYYYMMDD-SEQ` (contoh: `EQ-1-CNSD-20260519-001`)
- Radar: `RADAR-YYMMDD-SEQ` (contoh: `RADAR-260519-001`)
- Recorder: `RECORDER-YYMMDD-SEQ` (contoh: `RECORDER-260519-001`)
- AMSC: `AMSC-YYMMDD-SEQ` (contoh: `AMSC-260519-001`)
- Transmitter: `TRANSMITTER-YYMMDD-SEQ` (contoh: `TRANSMITTER-260519-001`)
- Receiver: `RECEIVER-YYMMDD-SEQ` (contoh: `RECEIVER-260519-001`)
- Glide Path: `GLIDEPATH-YYMMDD-SEQ` (contoh: `GLIDEPATH-260519-001`)
- Localizer: `LOCALIZER-YYMMDD-SEQ` (contoh: `LOCALIZER-260519-001`)

**Signature CNSD:** seluruh teknisi CNSD shift tersebut menjadi signer (per-row).
Manager dan supervisor sign di header record (nullable jika tidak ada di roster).

---

## 7. TFP Current State

**TFP Performance Check AOB Lantai Ground — ✅ Live (2026-05-19)**

| Kode | Modul | Form | Status | Route |
|---|---|---|---|---|
| TFP-001 | Performance Check AOB Lantai Ground | AOB-GROUND | ✅ Live | `/tfp/aob-ground` |
| TFP-002 | Performance Check AOB Lantai 1 & 2 | AOB-LT12 | ✅ Live | `/tfp/aob-lt12` |
| TFP-003+ | AOB Lantai lain, Transmitter, dll | — | 🚧 Coming Soon | — |

**Pola roster TFP:**
- `getShiftManager(shift, date)` → Manager Teknik (nullable).
- `getShiftSupervisorByDivision(shift, date, 'Support')` → Supervisor TFP (nullable).
- `getShiftPersonnel(shift, date)` filter `employee_type='Support'` → semua teknisi TFP.
- CNSD/CNS **tidak diikutkan**.
- Jika tidak ada teknisi TFP → create gagal 422.

**Format nomor form TFP:**
- AOB Ground: `TFP-AOBLTGND-YYMMDD-SEQ` (contoh: `TFP-AOBLTGND-260519-001`)

**Template AOB Ground:**
- 21 parameter (L1-N s/d Suhu Eq. Room). Row "Suhu Ruang ARO" TIDAK ADA.
- 17 fasilitas (Catu Daya Listrik s/d Door Lock).
- Disabled cells sesuai form resmi (is_disabled_map jsonb).
- Dropdown: Mode (Auto/Manual), Suplai Aktif (PLN/UPS, PLN 1/PLN 2), Kondisi (Baik/Normal/Tidak Baik).

**Signature TFP:** seluruh teknisi TFP shift tersebut menjadi signer (per-row).
Manager dan supervisor sign di header record (nullable jika tidak ada di roster).

---

## 8. TFP Next Development Pattern

**Next task: TFP manual** — ikuti pola CNSD manual, bukan dynamic form.

Perbedaan TFP vs CNSD:
| Aspek | CNSD | TFP |
|---|---|---|
| Teknisi | `employee_type='CNS'` | `employee_type='Support'` |
| Supervisor | `getShiftSupervisorByDivision(shift, date, 'CNS')` | `getShiftSupervisorByDivision(shift, date, 'Support')` |
| Manager Teknik | dari roster tanggal + shift | sama |
| Signature | per-row teknisi + header manager/supervisor | sama |
| Template | hardcoded per alat | hardcoded per alat |

Jangan mulai TFP sebelum membaca:
- `KIRO_TASK_CONTEXT.md` (file ini)
- `.agents/session-handoff.md`
- `.agents/instructions/cnsd-readiness-rules.md` (sebagai referensi pola)
- `BACKEND_CONTEXT.md`

---

## 8. Documentation Rules

Setiap task selesai wajib update:
- `.agents/session-handoff.md` — Current State terbaru
- Instruction terkait (mis. `tfp-rules.md` untuk TFP)
- `BACKEND_CONTEXT.md` — section modul baru
- `.kiro/steering/index.md` — integration status table
- `KIRO_TASK_CONTEXT.md` — jika ada perubahan pola besar

---

## 9. Grounding Current State

**Grounding Report — ✅ Live (2026-05-19)**

| Modul | Form | Status | Route |
|---|---|---|---|
| Laporan Grounding & Penangkal Petir | GROUNDING | ✅ Live | `/grounding` |

**Pola roster Grounding:**
- Identik dengan TFP: `employee_type='Support'`, Supervisor TFP, Manager Teknik
- Multiple records per shift diperbolehkan (berbeda peralatan)
- Jika tidak ada teknisi TFP → create gagal 422

**Format nomor laporan:**
- Grounding: `GROUNDING-YYMMDD-SEQ` (contoh: `GROUNDING-260519-001`)

**Template:**
- 6 VISUAL items (Terminal Udara, Konduktor Turun, Modul Penangkal Petir, Sambungan dan Clamp, Kabel Pembumian, Lightning Counter)
- 3 PENGUKURAN items (Nilai Tahanan Tanah, Nilai Tahanan Pentanahan, Uji Kontinuitas)

---

## 10. How to Use This Context

Template prompt pendek untuk task berikutnya:

```
Baca:
- KIRO_TASK_CONTEXT.md
- .agents/session-handoff.md
- .agents/instructions/[instruction-terkait].md
- BACKEND_CONTEXT.md
- file modul referensi yang relevan (mis. CnsdRadarMeterService.php untuk pola)

Ikuti semua hard constraints di KIRO_TASK_CONTEXT.md.
Gunakan RTK untuk semua terminal command.
Jangan menebak struktur file; ikuti pola aktual modul yang sudah ada.

Task:
[isi task spesifik]

Testing:
- rtk php artisan route:list
- rtk test php artisan test
- rtk npm run build
- rtk git diff

Update docs:
- session-handoff.md
- instruction terkait
- BACKEND_CONTEXT.md

Output:
- implementasi/akar masalah
- file diubah
- hasil test
- docs diupdate
- next task
```

---

## 10. Manual Form Development Hook

**Hook utama:** `.kiro/hooks/manual-form-workflow.md`

Wajib dibaca sebelum membuat form manual baru. Berlaku untuk:
- CNSD (EQ-1, Radar, Recorder, AMSC, dan form berikutnya)
- TFP (AOB Ground, AOB Lt 1 & 2, dan form berikutnya)
- Ground Check, Grounding, dan form manual lainnya

**Dynamic form tetap HOLD** — jangan diimplementasikan dalam kondisi apapun.

Hook mencakup: pull wajib, pola backend, pola frontend, roster, signature, nomor form, template item, UI detail, print view, testing, dokumentasi, commit, dan push.

---

## 11. Port Assignments (Local Dev)

| Service | Port |
|---|---|
| atoms-rostering backend | 8001 |
| atoms-rostering frontend | 5174 |
| atoms-maintenance backend | 8000 |
| atoms-maintenance frontend | 5173 |
