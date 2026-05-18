# TFP Performance Check — Module Rules

Last updated: 2026-05-19
Status: ✅ First TFP module live (Phase 5)

---

## Active Modules

| Code | Name | Form Type | Backend | Frontend | Print |
|---|---|---|---|---|---|
| TFP-001 | Performance Check AOB Lantai Ground | AOB-GROUND | ✅ | ✅ `/tfp/aob-ground` | ✅ |

All other TFP cards (AOB Lantai 1 & 2, Transmitter, Radar, Tower, VOR, Localizer, Glide Path)
remain **Coming Soon** and have no backend.

---

## Key Difference from CNSD

| Aspek | CNSD | TFP |
|---|---|---|
| Teknisi | `employee_type='CNS'` | `employee_type='Support'` |
| Supervisor | `getShiftSupervisorByDivision(shift, date, 'CNS')` | `getShiftSupervisorByDivision(shift, date, 'Support')` |
| Manager Teknik | dari roster tanggal + shift | sama |
| Signature | per-row teknisi + header manager/supervisor | sama |
| Template | hardcoded per alat | hardcoded per alat |

---

## Database

### `tfp_aob_ground_records`
Header per form. Satu record per (`form_type`, `date`, `shift_type`).
- Constraint unik dijaga oleh **partial unique index PostgreSQL**
  `tfp_aob_ground_records_unique_form_per_shift` yang ignore soft-deleted rows.
- `form_type` default `AOB-GROUND`.
- `day_name` — hari dalam bahasa Indonesia, di-set otomatis saat create dari `date`.
- `time_filled` — jam saat form **diisi** (HH:MM). **Refresh setiap kali user Simpan Perubahan**
  via `updateItems()` atau `updateFacilities()` di service. Setiap Simpan = snapshot waktu baru.
  Cocok dengan paper form yang mencatat "jam ketika petugas mengambil reading".
- `status`: `ongoing` | `on_hold` | `completed`.
- `manager_id`, `supervisor_id` nullable. Diisi otomatis dari roster saat create.
- Soft delete aktif.

### `tfp_aob_ground_technicians`
Snapshot teknisi TFP yang bertugas pada shift. Satu row per teknisi. Setiap row
punya signature kolom sendiri yang **immutable** dan **tidak boleh diwakilkan**.

### `tfp_aob_ground_items`
21 parameter form di-generate dari template saat record dibuat.
Kolom nilai: `panel_cos_a03_input/output`, `panel_ats_a12_input/output`,
`ups_tescom_a_input/output`, `ups_tescom_b_input/output`.
`is_disabled_map` (jsonb) menandai sel yang disabled/grey.

### `tfp_aob_ground_facilities`
17 fasilitas di-generate dari template saat record dibuat.
Kolom: `facility_name`, `kondisi` (Baik/Normal/Tidak Baik), `keterangan`.

---

## Item Template (AOB Ground)

Sumber kebenaran: `app/Services/Tfp/TfpAobGroundTemplate.php`.

**21 parameter:**

| No | Parameter | Unit |
|---|---|---|
| 1 | L1 - N | Volt |
| 2 | L2 - N | Volt |
| 3 | L3 - N | Volt |
| 4 | N - G | Volt |
| 5 | L1 - L2 | Volt |
| 6 | L1 - L3 | Volt |
| 7 | L2 - L3 | Volt |
| 8 | L1 | Ampere |
| 9 | L2 | Ampere |
| 10 | L3 | Ampere |
| 11 | N | Ampere |
| 12 | Frekuensi | Hz |
| 13 | Power Factor (Cos Θ) | — |
| 14 | Tegangan Battery | Volt |
| 15 | Arus Battery | Ampere |
| 16 | Kapasitas Battery | Ah |
| 17 | Suhu Battery | °C |
| 18 | Mode * | — |
| 19 | Suplai Aktif * | — |
| 20 | KWH Meter | — |
| 21 | Suhu Eq. Room | °C |

**Row "Suhu Ruang ARO" TIDAK ADA** — dihapus per permintaan user.

**Disabled cell rules (is_disabled_map):**
- Rows 1-12: semua 8 kolom enabled
- Row 13 (Power Factor): `ups_tescom_a_input/output`, `ups_tescom_b_input/output` disabled
- Rows 14-17 (Battery): `panel_cos_a03_input/output`, `panel_ats_a12_input/output` disabled
- Rows 18-19 (Mode, Suplai Aktif): `ups_tescom_a_input/output`, `ups_tescom_b_input/output` disabled
- Rows 20-21 (KWH Meter, Suhu Eq. Room): single value di `panel_cos_a03_input`, rest disabled

**17 fasilitas:**
Catu Daya Listrik, Penerangan, UPS Tescom A, UPS Tescom B,
AC 01-04 (Split Wall) Eq, AC 05 (Split Wall) Gd 21, AC 06 (Split Wall) Ex MCC,
AC 08 (Split Wall) ARO, Papan Nama AirNav, Atap, Plafond, Dinding, Pintu, Door Lock.

---

## Dropdown Rules

| Field | Options |
|---|---|
| Mode (Panel COS) | Auto / Manual |
| Mode (Panel ATS) | Auto / Manual |
| Suplai Aktif (Panel COS) | PLN / UPS |
| Suplai Aktif (Panel ATS) | PLN 1 / PLN 2 |
| Kondisi fasilitas | Baik / Normal / Tidak Baik |

---

## Roster Integration

Saat `POST /api/v1/tfp/aob-ground`:

1. Backend ambil **Manager Teknik** dari `RosteringIntegrationService::getShiftManager(date, shift)`.
2. Backend ambil **Supervisor TFP** dari `getShiftSupervisorByDivision(date, shift, 'Support')`.
3. Backend ambil **seluruh teknisi TFP** dari `getShiftPersonnel(date, shift)`
   yang `employee_type = 'Support'`. **Personel CNSD/CNS tidak diikutkan.**
4. Jika tidak ada teknisi TFP di shift → create return **422** dengan pesan jelas.
5. `time_filled` di-set ke `now()->format('H:i')` saat create.
6. `day_name` di-set ke nama hari Indonesia dari `date`.

---

## Signature Authorization

Identik dengan CNSD:
- Role check: Manager Teknik / Supervisor TFP / Teknisi TFP.
- Nama signer harus cocok dengan nama yang di-cache di record/row (tolerant compare).
- Untuk teknisi: sign per-row, tidak bisa mewakili teknisi lain.
- Signature **immutable**, **tidak boleh diwakilkan**.

HTTP status: 200 OK, 403 nama/role tidak cocok, 409 sudah ditandatangani, 422 payload invalid.

---

## Form Number Format

Format: `TFP-AOBLTGND-{YYMMDD}-{SEQ}`
Contoh: `TFP-AOBLTGND-260519-001`

- Prefix: `TFP-AOBLTGND`
- Tanggal: YYMMDD (6 digit)
- SEQ: 3-digit zero-padded, reset per tanggal
- Counter pakai `withTrashed()` agar soft-deleted rows tetap menambah seq.

---

## API Endpoints

Semua di bawah `/api/v1/tfp/aob-ground`, protected via `mockauth`:

| Method | URI | Description |
|---|---|---|
| GET    | `/api/v1/tfp/aob-ground` | Paginated list, filter `search`, `date`, `year`, `shift_type`, `status` |
| GET    | `/api/v1/tfp/aob-ground/years` | Daftar tahun tersedia |
| GET    | `/api/v1/tfp/aob-ground/template` | Template AOB Ground (parameters + facilities) |
| POST   | `/api/v1/tfp/aob-ground` | Create. Auto-generate items + facilities + auto-attach personnel. 409 jika sudah ada. |
| GET    | `/api/v1/tfp/aob-ground/{id}` | Detail dengan items + facilities + technicians + signatures |
| PUT    | `/api/v1/tfp/aob-ground/{id}` | Update item values + facility kondisi/keterangan |
| POST   | `/api/v1/tfp/aob-ground/{id}/sign` | Sign role `manager` / `supervisor` / `technician` |
| DELETE | `/api/v1/tfp/aob-ground/{id}` | Soft-delete. Role: Admin / MT |

Print: frontend-only (`/tfp/aob-ground/:id/print`).

---

## Frontend Architecture

| Page | Path | File |
|---|---|---|
| TFP index | `/tfp` | `pages/tfp/TfpIndexPage.tsx` |
| AOB Ground list | `/tfp/aob-ground` | `pages/tfp/TfpAobGroundListPage.tsx` |
| AOB Ground detail/edit | `/tfp/aob-ground/:id` | `pages/tfp/TfpAobGroundDetailPage.tsx` |
| AOB Ground print | `/tfp/aob-ground/:id/print` | `pages/tfp/TfpAobGroundPrintView.tsx` |
| Signature panel | — | `pages/tfp/components/TfpAobGroundSignaturePanel.tsx` |

Route print berada di luar `AppShell` (tidak ada topbar/sidebar) — identik dengan CNSD print views.

Type definitions: `src/types/tfpAobGround.ts`. API client: `src/services/tfpAobGroundService.ts`.

---

## Hard Constraints

- ❌ Jangan tambah backend untuk card TFP lain di session yang sama.
- ❌ Jangan biarkan UI menulis ke field signature secara langsung.
- ❌ Jangan ubah `tfp_aob_ground_*` schema tanpa migration baru.
- ❌ Jangan write ke `db_rostering`.
- ❌ Jangan ambil personel CNSD/CNS untuk TFP — hanya `employee_type='Support'`.
- ❌ Jangan ubah `form_number` / `date` / `shift_type` setelah tersimpan.
- ❌ Jangan biarkan teknisi A menandatangani row milik teknisi B.
- ❌ Jangan tambahkan row "Suhu Ruang ARO" — sudah dihapus per permintaan user.
- ✅ Item template hanya boleh diubah via `TfpAobGroundTemplate` source.
- ✅ Personnel selalu di-resolve dari roster, tidak dari mock data.
- ✅ CNSD dan Work Order tetap berjalan tanpa perubahan.

---

## What's NOT Built Yet

- Notification trigger saat create / completed (pola EQ-1 sudah ada, tinggal adopt).
- Backend untuk card TFP lain (AOB Lantai 1 & 2, Transmitter, dll) — masih Coming Soon.

---

## Print View Rules (2026-05-19 refinement)

Pola print view TFP mengikuti pola CNSD:

- **Tidak ada auto-print.** `window.print()` HANYA dipanggil saat user klik tombol "Print PDF". Jangan pakai `useEffect(() => window.print(), [])` atau `setTimeout(() => window.print(), ...)` karena React StrictMode menyebabkan effect dijalankan 2× di dev mode → dialog print muncul 2 kali.
- **Logo AirNav** — gunakan asset `/assets/icon/logoairnav.svg` (bukan placeholder). Pattern sama dengan CNSD/Work Order print view.
- **Single big table** — render parameter (kiri) + fasilitas (kanan) sebagai SATU `<table>` HTML, bukan dua tabel terpisah. Pakai `colSpan` untuk header (Panel COS/ATS/UPS span 2 kolom Input/Output) dan `rowSpan` untuk No/Parameter/Nama Fasilitas/Kondisi/Keterangan span 2 header rows.
- **Disabled cells** — render sebagai `<td className="bg-gray-300">` tanpa konten dan tanpa border-radius. Tampilkan sebagai blok grey solid yang menyatu dengan border tabel (merge visual).
- **CSS print** — `@media print { @page { size: A4 landscape; margin: 8mm; } body { background: white !important; } .print-hide { display: none !important; } }`.
- **Layout** — landscape A4, max-width 290mm.
- **Header layout:**
  - Kiri: logo AirNav + "AirNav Indonesia"
  - Tengah: judul "Performance Check AOB Lantai Ground"
  - Kanan: Perum LPPNPI / Cabang Surabaya / Teknik Fasilitas Penunjang
- **Footer signature (mirroring `CnsdReadinessPrintView`):**
  - Kiri (flex-1): tabel teknisi 3 kolom (No/Nama/Paraf). Saat belum sign tampilkan "Belum TTD" italic.
  - Tengah (w-24%): supervisor. Saat ada signature → image; saat ada nama tapi belum sign → dashed-border kotak kosong; saat tidak ada supervisor di shift → italic "Tidak ada supervisor pada shift ini".
  - Kanan (w-24%): manager teknik. Sama pattern dengan supervisor; saat tidak ditugaskan → italic "Manager Teknik tidak ditugaskan".

---

## Time Filled Rules (Jam Pelaksanaan)

`time_filled` mencatat **jam saat user mengisi form**, bukan jam saat record pertama kali dibuat:

- **Saat create record** — `time_filled = now()->format('H:i')` (set sekali).
- **Saat user Simpan Perubahan (`updateItems` atau `updateFacilities`)** — `time_filled` di-refresh ke `now()->format('H:i')`. Setiap Simpan = snapshot waktu baru. Implementasi: backend service `updateItems()` dan `updateFacilities()` menulis `$record->time_filled = now()->format('H:i'); $record->save();` di akhir transaction sebelum return fresh.
- **Tanggal dan hari (`date`, `day_name`)** TIDAK pernah di-refresh saat update — tetap dari record header (sesuai shift date).
- Format: `HH:mm` (mis. `20:15`).
- Frontend detail page menampilkan `record.time_filled` langsung. Setelah save, controller return detail record fresh → frontend `setRecord(updated)` → UI auto-update ke jam terbaru.

---

## Disabled Cell Enforcement

Disabled cell rules (`is_disabled_map` jsonb di `tfp_aob_ground_items`) di-enforce di **2 lapis**:

1. **Frontend detail page** — render disabled cells sebagai `<div className="bg-slate-300/80">` yang aria-hidden, tidak ada `<input>`. User tidak bisa mengetik atau memilih ke cell tersebut.
2. **Backend service `updateItems()`** — sebelum `fill()`, baca `is_disabled_map` per row dan strip kolom yang `true` dari payload. Bahkan jika klien mem-bypass UI dan mengirim payload `panel_cos_a03_input` untuk row Battery, backend tidak akan menulis ke kolom itu.

`is_disabled_map` adalah **source of truth** — di-set saat record dibuat dari `TfpAobGroundTemplate::parameters()` dan tidak boleh diubah lewat update endpoint.

---

## Time Filled Rules (Jam Pelaksanaan)

`time_filled` mencatat **jam saat user mengisi form**, bukan jam saat record pertama kali dibuat:

- **Saat create record** — `time_filled = now()->format('H:i')` (set sekali).
- **Saat user Simpan Perubahan (`updateItems` atau `updateFacilities`)** — `time_filled` di-refresh ke `now()->format('H:i')`. Setiap Simpan = snapshot waktu baru.
- **Timezone** — Backend `config/app.php` menggunakan `'timezone' => 'Asia/Jakarta'` (WIB). `now()->format('H:i')` menghasilkan WIB time yang benar. Jangan ubah ke UTC.
- Format: `HH:mm` (mis. `20:15`).
- Frontend detail page menampilkan `record.time_filled` langsung. Setelah save, controller return detail record fresh → frontend `setRecord(updated)` → UI auto-update ke jam terbaru.

---

## Form Layout Rules (Detail/Edit)

Detail/edit page memakai **DUA PANEL TERPISAH** (bukan satu unified table):

- **Kiri (flex-1)**: "Parameter Pengukuran" card dengan `<table>` 10 kolom (No, Parameter, COS×2, ATS×2, UPS A×2, UPS B×2).
  - Header row 1: `bg-sky-800` — No (rowSpan=2), Parameter (rowSpan=2), Panel COS (colSpan=2), Panel ATS (colSpan=2), UPS TESCOM A (colSpan=2), UPS TESCOM B (colSpan=2)
  - Header row 2: `bg-sky-700` — Input/Output × 4 panel groups
  - Rows 1-17: normal rendering dengan `CellInput` component
  - Row 18 (Mode): inline JSX, 6 `<td>` — No(1), Parameter(1), COS(colSpan=2 dropdown Auto/Manual), ATS(colSpan=2 dropdown Auto/Manual), UPS A(colSpan=2 grey), UPS B(colSpan=2 grey)
  - Row 19 (Suplai Aktif): inline JSX, 6 `<td>` — COS(colSpan=2 dropdown PLN/UPS), ATS(colSpan=2 dropdown PLN 1/PLN 2), UPS A/B grey
  - Rows 20-21 (KWH Meter, Suhu Eq. Room): inline JSX, 3 `<td>` — No(1), Parameter(1), single input (colSpan=8)
  - Disabled cells: `bg-slate-200` (lighter grey, tidak terlalu kontras)
- **Kanan (~380px)**: "Kondisi Fasilitas" card dengan `grid grid-cols-[1fr_90px_1fr]` rows.
  - Kondisi: dropdown Baik/Normal/Tidak Baik dengan emerald/red color coding
  - Keterangan: free text input
- **Row "Suhu Ruang ARO" tidak ada** — sudah dihapus per permintaan user.
- **Tidak ada teks "Mengikuti format form resmi"** — dihapus per permintaan user.
