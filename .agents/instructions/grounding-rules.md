# Grounding Report — Module Rules

Last updated: 2026-05-19
Status: ✅ Live

---

## Overview

Modul **Laporan Grounding & Penangkal Petir** adalah modul untuk mencatat hasil pemeriksaan sistem penangkal petir dan sistem pembumian di fasilitas AirNav Surabaya.

- Domain: TFP (Teknik Fasilitas Penunjang)
- Roster: teknisi TFP (employee_type='Support')
- Supervisor: TFP (getShiftSupervisorByDivision(..., 'Support'))
- Manager: Manager Teknik dari roster tanggal + shift
- Signature: per-row teknisi + header manager/supervisor (pola TFP AOB Ground)

---

## Key Differences from TFP AOB Ground

| Aspek | TFP AOB Ground | Grounding Report |
|---|---|---|
| Unique per shift | Ya (1 record per date+shift) | Tidak (multiple per shift, beda peralatan) |
| Template | 21 parameter + 17 fasilitas | 6 VISUAL + 3 PENGUKURAN items |
| Form number | `TFP-AOBLTGND-YYMMDD-SEQ` | `GROUNDING-YYMMDD-SEQ` |
| Kantor Unit Kerja | — | Default `Cabang Surabaya` (tidak bisa diubah user) |
| Equipment Name | — | Manual input wajib |
| Equipment Location | — | Manual input wajib |

---

## Database

### `grounding_report_records`
Header per laporan. Multiple records per (date, shift_type) diperbolehkan — berbeda peralatan.
- `report_number`: format `GROUNDING-YYMMDD-SEQ`
- `work_unit`: default `Cabang Surabaya` (tidak bisa diubah via UI)
- `equipment_name`: wajib diisi saat create
- `equipment_location`: wajib diisi saat create
- `day_name`, `time_filled`: otomatis dari backend
- `status`: `ongoing` | `on_hold` | `completed`
- Soft delete aktif

### `grounding_report_technicians`
Snapshot teknisi TFP per record. Per-row immutable signature.

### `grounding_report_items`
9 items dari template (6 VISUAL + 3 PENGUKURAN).
- `section_name`: `VISUAL` atau `PENGUKURAN`
- `availability`: `Ada` | `Tidak Ada` (VISUAL only, PENGUKURAN selalu null)
- `condition`: `Baik` | `Tidak Baik`
- `notes`: free text
- `standard`: null (VISUAL) atau `≤ 1 Ω` / `-` (PENGUKURAN)

---

## Item Template

Sumber: `app/Services/Grounding/GroundingReportTemplate.php`.

### VISUAL (6 items)
1. Terminal Udara
2. Konduktor Turun
3. Modul Penangkal Petir
4. Sambungan dan Clamp
5. Kabel Pembumian
6. Lightning Counter

### PENGUKURAN (3 items)
1. Nilai Tahanan Tahanan Tanah — standard: `≤ 1 Ω`
2. Nilai Tahanan Pentanahan Peralatan — standard: `≤ 1 Ω`
3. Uji Kontinuitas Konduktor Turun dan Kabel Pentanahan — standard: `-`

---

## Roster Integration

Saat `POST /api/v1/grounding/reports`:
1. Backend ambil Manager Teknik dari `getShiftManager(date, shift)`.
2. Backend ambil Supervisor TFP dari `getShiftSupervisorByDivision(date, shift, 'Support')`.
3. Backend ambil seluruh teknisi TFP dari `getShiftPersonnel(date, shift)` filter `employee_type = 'Support'`.
4. Jika tidak ada teknisi TFP → create return **422** dengan pesan jelas.
5. `time_filled` di-set ke `now()->format('H:i')` saat create, refresh setiap Simpan Perubahan.

---

## Signature Authorization

Identik dengan TFP AOB Ground:
- Role check: Manager Teknik / Supervisor TFP / Teknisi TFP.
- Nama signer harus cocok dengan nama yang di-cache di record/row (tolerant compare).
- Untuk teknisi: sign per-row, tidak bisa mewakili teknisi lain.
- Signature **immutable**, **tidak boleh diwakilkan**.

HTTP status: 200 OK, 403 nama/role tidak cocok, 409 sudah ditandatangani, 422 payload invalid.

---

## Form Number Format

Format: `GROUNDING-{YYMMDD}-{SEQ}`
Contoh: `GROUNDING-260519-001`

- Prefix: `GROUNDING`
- Tanggal: YYMMDD (6 digit)
- SEQ: 3-digit zero-padded, reset per tanggal
- Counter pakai `withTrashed()` agar soft-deleted rows tetap menambah seq.
- Multiple records per tanggal diperbolehkan (berbeda peralatan)

---

## API Endpoints

Semua di bawah `/api/v1/grounding/reports`, protected via `mockauth`:

| Method | URI | Description |
|---|---|---|
| GET    | `/api/v1/grounding/reports` | Paginated list, filter `search`, `date`, `year`, `shift_type`, `status` |
| GET    | `/api/v1/grounding/reports/years` | Daftar tahun tersedia |
| GET    | `/api/v1/grounding/reports/template` | Template checklist (sections + items) |
| POST   | `/api/v1/grounding/reports` | Create. Auto-generate items + auto-attach personnel. |
| GET    | `/api/v1/grounding/reports/{id}` | Detail dengan items + technicians + signatures |
| PUT    | `/api/v1/grounding/reports/{id}` | Update item values (availability, condition, notes) |
| POST   | `/api/v1/grounding/reports/{id}/sign` | Sign role `manager` / `supervisor` / `technician` |
| DELETE | `/api/v1/grounding/reports/{id}` | Soft-delete. Role: Admin / MT |

Print: frontend-only (`/grounding/reports/:id/print`).

---

## Frontend Architecture

| Page | Path | File |
|---|---|---|
| Grounding list | `/grounding` | `pages/grounding/GroundingIndexPage.tsx` |
| Grounding detail/edit | `/grounding/reports/:id` | `pages/grounding/GroundingReportDetailPage.tsx` |
| Grounding print | `/grounding/reports/:id/print` | `pages/grounding/GroundingReportPrintView.tsx` |
| Signature panel | — | `pages/grounding/components/GroundingReportSignaturePanel.tsx` |

Type definitions: `src/types/grounding.ts`. API client: `src/services/groundingReportService.ts`.

---

## UI Rules

- **Kantor Unit Kerja**: tidak ada dropdown — default `Cabang Surabaya`, tampil readonly
- **Nama Peralatan + Lokasi Peralatan**: manual input wajib saat create (via modal)
- **Dibuat Oleh / Disetujui Oleh**: TIDAK ADA — diganti dengan signature panel TFP-style
- **Tanggal + Jam**: otomatis dari backend (time_filled refresh setiap Simpan Perubahan)
- **Create flow**: klik Tambah Laporan → modal isi nama + lokasi → backend create → navigate ke detail page
- **Print**: manual only, no auto-print

---

## Print View Rules

Mengikuti pola TFP AOB Ground:
- Header: AirNav logo kiri, Perum LPPNPI kanan
- Judul: "Checklist Fasilitas dan Peralatan / Pemeliharaan Sistem Penangkal Petir dan Sistem Pembumian"
- Info: No. Laporan, Tanggal, Jam, Shift, Kantor Unit Kerja, Nama Peralatan, Lokasi Peralatan
- Section VISUAL table (No, Item, Ketersediaan, Kondisi, Catatan)
- Section PENGUKURAN table (No, Item, Standard, Kondisi, Catatan)
- Footer: Waktu Pelaksanaan (Hari/Tanggal/Jam) + Pelaksana Teknisi | Supervisor | Manager Teknik
- Print manual only (button click)

---

## Hard Constraints

- ❌ Jangan tambah dropdown Kantor Unit Kerja — selalu Cabang Surabaya
- ❌ Jangan buat field "Dibuat Oleh" / "Disetujui Oleh" — pakai signature panel
- ❌ Jangan ambil personel CNSD/CNS — hanya `employee_type='Support'`
- ❌ Jangan biarkan UI menulis ke field signature secara langsung
- ❌ Jangan ubah `grounding_report_*` schema tanpa migration baru
- ❌ Jangan write ke `db_rostering`
- ✅ Multiple records per shift diperbolehkan (berbeda peralatan)
- ✅ Item template hanya boleh diubah via `GroundingReportTemplate` source
- ✅ Personnel selalu di-resolve dari roster, tidak dari mock data
- ✅ CNSD, TFP, dan Work Order tetap berjalan tanpa perubahan
