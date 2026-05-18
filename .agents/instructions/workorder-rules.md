# Work Order Rules

Last updated: 2026-05-15

## Status Values

The only valid Work Order statuses are:
- `ongoing`
- `on_hold`
- `completed`

Status is derived by backend logic. Clients must not manually set status during create, update, or signing.

## Status Transition Logic

- `ongoing`: required signatures are incomplete and the shift has not ended.
- `on_hold`: required signatures are incomplete and the shift has ended.
- `completed`: all required signatures are present.

The implementation uses `HasSignature::recalculateStatus()` through the Work Order model.
`WorkOrder::isShiftEnded()` menggunakan `RosteringIntegrationService::isShiftEnded()` dengan fallback hardcoded jika rostering DB tidak tersedia:
- `pagi`: 13:00
- `siang`: 19:00
- `malam`: 07:00 hari berikutnya

## WO Number Format

Format resmi: `WO-{DIVISI}-{YYYYMMDD}-{SEQ}`

Contoh:
- `WO-CNSD-20260516-001`
- `WO-TFP-20260516-001`

Aturan:
- `YYYYMMDD` = tanggal pembuatan WO (dari `now()->format('Ymd')`).
- `SEQ` = 3-digit zero-padded, reset per tanggal + divisi.
- Format lama `WO-{DIV}-{DD}-{MM}-{YYYY}-{SEQ}` masih bisa ditampilkan dan dicari (ILIKE search).
- Jangan mengubah nomor WO yang sudah tersimpan.

Implementasi: `WorkOrderService::generateWoNumber(string $division): string`.

## Work Order List Filter

**Backend endpoint:** `GET /api/v1/work-orders` dengan query params:

| Param | Deskripsi |
|---|---|
| `search` | ILIKE pada `wo_number` dan `description` |
| `shift_date` | Exact match `YYYY-MM-DD` |
| `year` | Filter tahun via `EXTRACT(YEAR FROM shift_date)` |
| `division` | Exact match (`CNSD`, `TFP`) |
| `shift_type` | Exact match (`pagi`, `siang`, `malam`) |
| `status` | Exact match (`ongoing`, `on_hold`, `completed`) |

**Endpoint tambahan:** `GET /api/v1/work-orders/years` — array tahun tersedia dari data, descending, selalu menyertakan tahun berjalan.

**Frontend:** `WorkOrderListPage.tsx`:
- Debounce search 350ms.
- Semua filter dikirim ke backend saat berubah.
- Active filter chips (badge per filter aktif, bisa close satu per satu).
- Tombol Reset Filter di samping search bar.
- Result count di kanan bawah filter bar.
- Empty state dua varian: DB kosong vs filter no-result.

Semua pengambilan personel dari rostering wajib difilter berdasarkan **tanggal + shift secara eksplisit**, bukan tanggal saja dan bukan time-based auto-detect di backend.

Alasan: backend Laravel berjalan di timezone `UTC`. Auto-detect "shift sekarang" lewat `Carbon::now()` di backend menghasilkan shift yang salah untuk operasional WIB (UTC+7). Frontend harus menghitung shift dari client clock dan mengirimkannya eksplisit.

Helper frontend: `src/lib/shiftUtils.ts`
- `getCurrentShiftType()` → `'pagi' | 'siang' | 'malam'`
- `getCurrentShiftDate()` → `'YYYY-MM-DD'` di local time, dengan koreksi malam shift (00:00–06:59 dipetakan ke tanggal sebelumnya)
- `getShiftLabel()` → label, jam mulai/selesai, emoji

Endpoint backend:
- `GET /api/v1/personnel/shift-today?date=YYYY-MM-DD&shift_type=pagi|siang|malam`
- Selalu kirim **kedua** parameter dari frontend.

Aturan pemakaian per layar:

| Layar | Tanggal | Shift |
|---|---|---|
| Dashboard "Personel Bertugas" | `getCurrentShiftDate()` (today di WIB) | `getCurrentShiftType()` |
| Form Buat Work Order | `getCurrentShiftDate()` | `getCurrentShiftType()` |
| Form Edit Work Order | `workOrder.shift_date` | `workOrder.shift_type` |
| Work Order Personal (existing) | unchanged — sudah benar, **jangan disentuh** |  |

Response endpoint mengandung:
- `manager` — Manager Teknik untuk shift+tanggal tersebut (atau `null`)
- `supervisor_cnsd` — CNS dengan grade ≥ 13 (atau `null`)
- `supervisor_tfp` — Support dengan grade ≥ 13 (atau `null`)
- `supervisor` — alias kompatibilitas: CNSD diutamakan, fallback TFP
- `personnel` — semua CNS + Support yang bertugas di shift+tanggal tersebut
- `has_supervisor`, `roster_available`, `shift_times`

## Signature Columns

Work Order signatures are stored directly on `work_orders`:

| Role | Name Column | Signature Column | Signer Column | Timestamp Column |
|------|-------------|------------------|---------------|------------------|
| MT | `mt_name` | `mt_signature` | `mt_signed_by` | `mt_signed_at` |
| Supervisor | `supervisor_name` | `supervisor_signature` | `supervisor_signed_by` | `supervisor_signed_at` |
| Technician | `technician_name` | `technician_signature` | `technician_signed_by` | `technician_signed_at` |

Signatures are base64 PNG data URLs. They are immutable after being saved.

## Required Signatures

When `has_supervisor = true`:
- `mt`
- `supervisor`
- `technician`

When `has_supervisor = false`:
- `mt`
- `technician`

Supervisor signature is not required when `has_supervisor = false`, and supervisor signature columns remain null.

## has_supervisor Logic

`has_supervisor` is set at Work Order creation time.

Current implementation (✅ resolved 2026-05-15):
- `WorkOrderService::createWorkOrder()` memanggil `resolveShiftPersonnelFromRostering()`.
- Jika roster dipublish: MT dan supervisor di-resolve otomatis dari rostering. `has_supervisor` di-set berdasarkan kehadiran supervisor di shift.
- Jika roster belum dipublish: fallback ke data dari request (`supervisor_id` present → `has_supervisor = true`).

Jangan ubah logic ini tanpa instruksi eksplisit.

## Technician Round-Robin Selection

At Work Order creation:
1. `resolveShiftPersonnelFromRostering()` di `WorkOrderService` query technicians dari rostering DB via `RosteringIntegrationService::getShiftPersonnel()`.
2. Count existing Work Orders for the same `division`, `shift_date`, `shift_type`, and `shift_id` when present.
3. Select technician at `record_count % total_technicians`.
4. Cache the selected technician name in `technician_name`.
5. Store selected technician ID in `assigned_technician_id`.

This selection is final and must not be changed after creation.
Jika roster belum dipublish, fallback ke `personnel` dari request.

## Name Caching

These names are cached at creation time:
- `mt_name`
- `supervisor_name`
- `technician_name`
- legacy snapshots: `manager_name_snapshot`, `supervisor_name_snapshot`

Cached names preserve the printed record even if rostering data changes later.

## Signing API

Endpoint:
- `POST /api/v1/work-orders/{id}/sign`

Body:
```json
{
  "role": "mt",
  "signature": "data:image/png;base64,..."
}
```

Allowed roles:
- `mt`
- `supervisor`
- `technician`

Validation and business rules:
- Authenticated user role must match the submitted signing role.
- **Signer name must match the cached signer name on the Work Order** (tolerant compare: trim + collapse whitespace + case-insensitive). Implementasi di `WorkOrderService::assertSignerCanSignRole()` memakai helper `WorkOrderService::namesMatch()`.
- Untuk `technician`, jika nama tidak cocok dengan `technician_name`, fallback diperbolehkan: nama signer cocok dengan salah satu nama di `personnel[]`. Tetap harus role `Teknisi CNSD/TFP`.
- Completed Work Orders cannot be signed.
- Existing signatures cannot be overwritten.
- Signature must start with `data:image/png;base64,`.
- Base64 content must be valid and non-empty.

HTTP status mapping:
- `200` — signature saved.
- `403` — wrong role atau wrong name (user mencoba menandatangani role yang bukan miliknya). Backend melempar `App\Exceptions\SignerNotAuthorizedException` dengan pesan `Tanda tangan hanya dapat dilakukan oleh {nama yang berwenang}. Tidak boleh diwakilkan.`.
- `409` — sudah pernah ditandatangani (immutable) atau Work Order sudah `completed`.
- `422` — payload signature tidak valid (bukan PNG base64).

## Print Data Structure

Endpoint:
- `GET /api/v1/work-orders/{id}/print`

Response `data` contains:
- `work_order`: full Work Order record
- `required_signatures`
- `pending_signatures`

The `work_order.signatures` object includes `mt`, `supervisor`, and `technician` signature data.

## Print Layout Rules

| Bagian | Aturan |
|---|---|
| Header | Logo PERUM LPPNPI cabang Surabaya + judul + nomor WO |
| Tertuju | Divisi tujuan WO |
| Tabel Personel Shift | **Hanya kolom No dan Nama**. Kolom Role dihapus. |
| Output (checklist 4 opsi) | Pakai centang `✓` di dalam kotak, **bukan** `X` |
| Pelaksanaan (Selesai / Dilanjutkan / Tidak Bisa) | Pakai centang `✓` di dalam kotak, **bukan** `X` |
| Tanda tangan | 3 kolom: Manager Teknik, Supervisor, Teknisi. Jika `has_supervisor = false`, kolom Supervisor menampilkan "Tidak Ada". |

## Dashboard "Personel Bertugas"

Layar dashboard menampilkan personel berdasarkan tanggal + shift aktif (lihat *Roster Filter Rule*) dalam tiga blok berurutan:

1. **MANAGER TEKNIK** — `manager` dari endpoint shift-today. Jika `null`, tampilkan empty state: *"Manager Teknik tidak tersedia pada shift ini"*.
2. **DIVISI CNSD** — `supervisor_cnsd` (jika ada) + semua personnel dengan `employee_type = 'CNS'`.
3. **DIVISI TFP** — `supervisor_tfp` (jika ada) + semua personnel dengan `employee_type = 'Support'`.

Manager Teknik dipilih dari `shift_assignments` di mana `employees.employee_type = 'Manager Teknik'`. Tabel `manager_duties` di rostering belum dipakai (kosong di data live), jadi resolver MT diturunkan dari shift_assignments path yang sama dengan personnel biasa.

Supervisor tidak boleh tampil ganda (sebagai supervisor dan sebagai teknisi). Manager juga tidak boleh tampil di blok divisi. Filter di frontend membuang user_id yang sudah ditampilkan sebagai supervisor/manager dari list teknisi.

Jika `roster_available = false`, tampilkan pesan "Roster belum dipublish untuk shift ini".

## Form Buat / Edit Work Order

| Kondisi | Source roster | Manager | Supervisor | Teknisi |
|---|---|---|---|---|
| Buat baru, roster ada | `getCurrentShiftDate()` + `getCurrentShiftType()` | dari `manager` (auto-resolve backend) | `supervisor_cnsd` jika divisi CNSD, `supervisor_tfp` jika divisi TFP (auto-resolve backend) | semua personnel dengan `employee_type` cocok dengan divisi (auto-fill backend bila kosong) |
| Buat baru, roster kosong | — | mock fallback | mock fallback | mock fallback |
| Edit | `workOrder.shift_date` + `workOrder.shift_type` | dari roster historis | sama, per divisi | sama, per divisi |

Saat user memilih divisi (CNSD/TFP), form **harus langsung** menampilkan daftar teknisi divisi tersebut pada shift+tanggal yang aktif. Tidak menunggu submit.

Badge roster aktif harus pakai karakter Unicode bersih: `●` (bullet), bukan mojibake `â—`.

### Identifier User

Frontend mengirim **rostering_user_id** untuk semua field user (manager_id, supervisor_id, assigned_technician_id, personnel[].user_id) — itu ID yang dikenal dari endpoint `/api/v1/personnel/shift-today`. Backend `WorkOrderService::mapPayloadRosteringIdsToLocal()` mentranslasikannya ke `local_users.id` dan membuat row local_users baru lewat `LocalUserResolver::ensureLocalUser()` jika user belum pernah di-cache.

FormRequest validation tidak menggunakan `exists:local_users,id` untuk field user-id karena resolver akan lazy-create.

### Personnel Auto-Fill (Shift WO)

Untuk `wo_type = 'shift'`, `personnel` array di payload bersifat **opsional**. Jika kosong, backend `WorkOrderService::autoFillShiftPersonnelFromRostering()` mengisi otomatis dari rostering shift personnel yang `employee_type` cocok dengan divisi WO. Ini memastikan create tidak silent-fail saat roster sementara unreachable di sisi frontend.

Untuk `wo_type = 'personal'`, frontend wajib mengirim `assigned_technician_id` (rostering_user_id) — backend tidak meng-auto-fill teknisi personal.

### Frontend Validation Sebelum Submit

`WorkOrderFormModal.handleSubmit()` melakukan pre-validation client-side untuk mencegah silent 422:

| Field | Aturan |
|---|---|
| `division` | wajib (create only) |
| `description` | wajib, minimal 10 karakter (trim) |
| `output_types` | wajib, minimal 1 |
| `output_other` | wajib bila `outputs.includes('other')` |
| `selectedTechnician` | wajib bila `wo_type='personal'` (create only) |

Pesan error ditampilkan di banner `<div role="alert">` di atas form, di-scroll ke tengah viewport via `scrollIntoView` agar terlihat tanpa scroll manual.

Submit handler juga menulis safe payload summary ke console saat `import.meta.env.DEV` true (tanpa token / signature).

### Edit — Field yang Boleh Diubah

Hanya field berikut yang boleh diubah saat edit:
- `description`
- `output_types` (dan `output_other` jika ada `'other'`)
- `start_time`, `end_time`
- `completion_status`
- `notes_kendala`, `notes_usulan`, `notes_pemberi_tugas`

Field berikut **tidak boleh** diubah saat edit:
- `wo_type`, `division`, `shift_type`, `shift_date`
- `manager_id`, `supervisor_id`, `assigned_technician_id`
- `manager_name_snapshot`, `supervisor_name_snapshot`, `mt_name`, `supervisor_name`, `technician_name`
- semua field signature
- `status` (di-derive dari signature dan shift, bukan diset client)
- `personnel[]`

Frontend `WorkOrderFormModal` di mode edit mengunci radio Tipe WO, dropdown Divisi, dan dropdown Teknisi. Mode edit fetch WO real dari API (dengan fallback ke mock) — tidak lagi bergantung pada `mockWorkOrders.find()` yang dulu menyebabkan silent fail untuk WO yang hanya ada di DB.

### Detail Page Refresh

`WorkOrderDetailPage` melakukan `fetchWorkOrder()` pada mount, pada `window 'focus'` event, dan setelah setiap mutation (sign / feedback). Mutasi tidak pernah hanya percaya response — selalu refetch supaya state pada layar canonical.
