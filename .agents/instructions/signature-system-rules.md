# Signature System Rules

Last updated: 2026-05-19

## Scope

The global signature system applies to:
- Work Orders ✅ Implemented
- CNSD Equipment Readiness — Form EQ-1 ✅ Implemented (Phase 4 pilot, 2026-05-17)
- CNSD Radar Meter Reading ✅ Implemented (Phase 4 Module 2, 2026-05-18)
- CNSD Recorder Meter Reading ✅ Implemented (Phase 4 Module 3, 2026-05-19)
- Meter Reading TFP — Planned (next)
- Ground Check — Planned
- Grounding — Planned
- Logbook — Planned
- Reporting — Planned

Future modules must reuse the same trait and API pattern.

## Storage Rules

- Signatures are hand-drawn canvas images.
- The frontend sends signatures as base64 PNG data URLs.
- Signatures are stored directly in database long text fields.
- No file upload is allowed.
- No filesystem storage is allowed.
- No external hosting is allowed.
- Once saved, a signature cannot be overwritten or deleted.
- Admin override is out of scope.

## HasSignature Trait

Location:
- `atoms-maintenance/backend_atoms-maintenance/app/Traits/HasSignature.php`

Methods:
- `saveSignature(string $role, string $base64, int $userId): void`
- `getRequiredSignatures(): array`
- `isComplete(): bool`
- `getPendingSignatures(): array`
- `recalculateStatus(): string`

Behavior:
- Validates signature prefix `data:image/png;base64,`.
- Validates non-empty base64 payload.
- Rejects invalid base64.
- Rejects unsupported roles.
- Rejects overwrite attempts.
- Saves signature, signer ID, signer timestamp, and signer name when available.
- Returns status as `completed` when all required signatures exist.
- Returns `on_hold` when required signatures are missing and the model shift has ended.
- Returns `ongoing` otherwise.

Models may override:
- `requiredSignatureRoles()`
- `signatureRoleMap()`
- `isShiftEnded()`

## Canvas-to-Base64 Flow

1. User draws signature on frontend canvas.
2. Frontend converts canvas to PNG data URL.
3. Frontend posts the data URL to module sign endpoint.
4. Backend validates data URL format and base64 payload.
5. Backend stores the string directly in the module table.
6. Print views read the base64 string directly from API response.

## Signing API Pattern

Pattern:
- `POST /api/v1/{module}/{id}/sign`

Body:
```json
{
  "role": "mt",
  "signature": "data:image/png;base64,..."
}
```

Success response:
```json
{
  "success": true,
  "message": "Signature saved successfully",
  "data": {
    "signed_role": "mt",
    "pending_roles": ["supervisor", "technician"],
    "current_status": "ongoing",
    "record": {}
  },
  "errors": null
}
```

Validation rules:
- User must be authenticated.
- Authenticated user's role must match submitted signing role (e.g. role `mt` → user.role = `Manager Teknik`).
- **Authenticated user's name must match the cached signer name on the record** (e.g. `mt_name`, `supervisor_name`, `technician_name`). Comparison is tolerant: trim, collapse whitespace, case-insensitive. Helper di `WorkOrderService::namesMatch()`.
- Untuk role `technician` di Work Order Shift, fallback diperbolehkan: nama signer cocok dengan salah satu nama di `personnel[]`.
- Tidak boleh diwakilkan: walaupun role cocok, signer dengan nama berbeda akan ditolak 403.
- Record must not be completed.
- Signature field for the role must be null before signing.
- Signature must start with `data:image/png;base64,`.
- Base64 content must be valid and non-empty.

HTTP status mapping:
- `200` — signature saved.
- `403` — name mismatch atau role mismatch (`SignerNotAuthorizedException`).
- `409` — sudah pernah ditandatangani atau record sudah `completed`.
- `422` — payload tidak valid.

Frontend boleh menambahkan UI:
- Tombol Tanda Tangan hanya aktif bila `currentUser.role` sesuai dan `namesMatch(cachedName, currentUser.name)`.
- Jika tidak berhak, tampilkan teks: *"Tanda tangan hanya dapat dilakukan oleh {nama yang berwenang}"*.
- Validasi utama tetap di backend.

## Round-Robin Technician Selection

When a new shift-based record is created:
1. Query technicians for the shift dari rostering menggunakan read-only connection (`DB::connection('rostering')`).
2. Count existing module records for that same shift.
3. Select technician at `record_count % total_technicians`.
4. Cache selected technician name immediately.
5. Store selected technician ID/reference.
6. Do not change the selected technician later.

Work Order sudah menggunakan `RosteringIntegrationService` untuk resolve shift personnel dari rostering DB.
Modul lain (CNSD, TFP, dll) harus menggunakan pattern yang sama saat diimplementasi.

## has_supervisor Logic

Modules with supervisor signatures must set `has_supervisor` at creation.

If `has_supervisor = true`:
- Supervisor name is cached.
- Supervisor signature is required.
- Print view shows supervisor column with data.

If `has_supervisor = false`:
- Supervisor columns remain null.
- Supervisor signature is not required.
- Print view shows supervisor column empty.

`has_supervisor` harus di-derive dari kehadiran supervisor di shift rostering.
Work Order sudah menggunakan `RosteringIntegrationService::getShiftSupervisor()` untuk ini.
Modul lain harus menggunakan pattern yang sama.

## Required Signatories

| Module | Required Signatures |
|--------|---------------------|
| Work Orders | MT, Supervisor*, Technician |
| CNSD Equipment Readiness (EQ-1) | Manager**, Supervisor CNSD**, **every** Technician on duty (per-row) |
| CNSD Radar Meter Reading | Manager**, Supervisor CNSD**, **every** Technician on duty (per-row) |
| CNSD Recorder Meter Reading | Manager**, Supervisor CNSD**, **every** Technician on duty (per-row) |
| Meter Reading TFP | Manager**, Supervisor TFP**, **every** TFP Technician on duty (per-row) — Planned |
| Ground Check | Technician |
| Grounding | MT, Technician |
| Logbook | Technician |
| Reporting | MT, Supervisor* |

`*` means only required when `has_supervisor = true`.
`**` for CNSD/TFP modules, manager and supervisor are only required when their
columns are populated at create-time (resolved from rostering). When the roster
has no MT/Supervisor for the shift, those signatures are skipped — but **every**
technician row must still be signed for the record to reach `completed`.

## Module Role Keys

Canonical role keys:
- `mt`              — Manager Teknik (Work Order)
- `manager`         — Manager Teknik (CNSD readiness — same person, different key)
- `supervisor`      — Supervisor (Work Order, CNSD readiness)
- `technician`      — Teknisi (Work Order single-row, CNSD readiness per-row)
- `cnsd_officer`    — Reserved for legacy meter-reading flow
- `tfp_officer`     — Reserved for legacy meter-reading flow

Accepted aliases:
- `cnsd` maps to `cnsd_officer`
- `tfp`  maps to `tfp_officer`

Why two keys for the same human role (`mt` vs `manager`)? Each module owns its
own role-map (via `signatureRoleMap()` override on the trait). Work Orders
historically used `mt`; CNSD readiness uses `manager` for clarity since the
column on the table is `manager_*`. Both behave identically.
