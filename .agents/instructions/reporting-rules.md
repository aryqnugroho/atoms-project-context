# Reporting / Laporan Kerusakan — Module Rules

> Domain: Reporting (Damage Report)
> Status: ✅ Live (2026-05-19)
> Top-level route: `/reporting`
> Backend prefix: `/api/v1/reporting/damage-reports` and `/api/v1/reporting/personnel`

---

## 1. Cakupan & Pembeda

Reporting **berbeda** dari Work Order, CNSD, TFP, Ground Check, dan Grounding.

Reporting adalah dokumen laporan kerusakan formal. **Tidak terikat shift** dan **tidak otomatis** mengambil personel dari rostering.

| Aspek | Reporting | Work Order / CNSD / TFP / Grounding |
|------|----------|-------------------------------------|
| Roster | ❌ Tidak dipakai | ✅ Wajib (date + shift) |
| Manager Teknik | Manual select | Auto-resolve dari roster |
| Pelaksana | Manual select (campur CNSD/TFP) | Auto-resolve dari roster shift |
| Shift | Tidak ada konsep shift | Wajib pagi/siang/malam |
| Form | Dokumen formal cetak | Form/checklist operasional |

---

## 2. Format Nomor Surat

Format: `LTK-YYMMDD-SEQ`
Contoh: `LTK-260519-001`, `LTK-260519-002`

- Prefix `LTK` (Laporan Teknik Kerusakan)
- `YYMMDD` = 2-digit tahun + bulan + tanggal
- `SEQ` = 3-digit zero-padded, reset per tanggal
- Counter pakai `withTrashed()` agar soft-deleted rows tetap menambah sequence
- Tampil di list, detail, dan print view

---

## 3. Database

### `reporting_damage_reports`
Header laporan kerusakan. Field utama:
- `report_number` (unique, format LTK-YYMMDD-SEQ)
- `report_date` (date)
- `day_name` (Indonesian day name, derived from date)
- `location`, `facility`, `equipment_name`, `equipment_module`
- `damage_category` enum: `Ringan` | `Sedang` | `Berat`
- `damage_description` text (required)
- `damage_cause` text (nullable)
- `repair_action` text (nullable)
- `repair_by_type` enum: `lokasi` | `pusat` (nullable)
- `damage_started_at`, `repair_finished_at` datetime (nullable)
- `downtime_hours` decimal (nullable, auto-calc from delta jika kosong)
- `obstacle_code` (nullable, AU/PK/TT/SC/TR/ST/PC/AL/TH)
- `obstacle_description` (nullable, **wajib** jika obstacle_code = AL)
- `status` enum: `ongoing` | `on_hold` | `completed`
- Manager: `manager_id`, `manager_name`, `manager_role`, `manager_signature`, `manager_signed_by`, `manager_signed_at`
- Audit: `created_by_id`, `created_by_name`, `timestamps`, `softDeletes`

### `reporting_damage_repairers`
Per-row pelaksana perbaikan + signature immutable. Field utama:
- `report_id` (FK cascadeOnDelete)
- `person_id` (FK local_users, nullable)
- `person_name`, `person_role`, `person_division`
- `signature` (longText, nullable, base64 PNG)
- `signed_by` (FK local_users, nullable)
- `signed_at` (timestamp, nullable)
- `sort_order`

---

## 4. Personnel Selector

### `GET /api/v1/reporting/personnel?scope=manager&search=...`
Returns: hanya personel dengan `role = "Manager Teknik"` dari `local_users` aktif.

### `GET /api/v1/reporting/personnel?scope=repairer&search=...&division=CNSD|TFP`
Returns: gabungan personel dengan role berikut (dari `local_users` aktif):
- Teknisi CNSD
- Teknisi TFP
- Supervisor CNSD
- Supervisor TFP

Filter `division` opsional. Filter `search` ILIKE pada nama.

**Catatan:**
- Source: `local_users` cache (tidak query rostering shift).
- Tidak ada filter `employee_type` atau shift.
- Boleh campur CNSD/TFP, teknisi/supervisor.

---

## 5. Endpoint API

Semua endpoint protected dengan middleware `mockauth`.

| Method | URI | Roles |
|---|---|---|
| GET    | `/api/v1/reporting/damage-reports` | semua authenticated |
| GET    | `/api/v1/reporting/damage-reports/years` | semua authenticated |
| POST   | `/api/v1/reporting/damage-reports` | Admin / MT / Sup CNSD/TFP / Tek CNSD/TFP |
| GET    | `/api/v1/reporting/damage-reports/{id}` | semua authenticated |
| PUT    | `/api/v1/reporting/damage-reports/{id}` | semua authenticated |
| POST   | `/api/v1/reporting/damage-reports/{id}/sign` | sesuai role + name match |
| DELETE | `/api/v1/reporting/damage-reports/{id}` | Admin / Manager Teknik |
| GET    | `/api/v1/reporting/personnel` | semua authenticated |

Total: 8 routes (7 damage-reports + 1 personnel selector).

Filter parameter `index`: `search`, `date`, `year`, `damage_category`, `obstacle_code`, `status`, `sort_by`, `sort_dir`, `per_page`.

---

## 6. Validation Rules

### Create
- `report_date` required, valid date.
- `location`, `facility`, `equipment_name` required.
- `damage_category` required, in [`Ringan`, `Sedang`, `Berat`].
- `damage_description` required.
- `obstacle_code` nullable; jika `AL`, `obstacle_description` **required**.
- `manager_id` required, exists di `local_users`. Backend juga verifikasi `role = Manager Teknik`.
- `repairers` required, min 1 item.
- `repairers.*.person_name` required, max 120.
- Tidak boleh duplikat `person_id` di payload.

### Update
- Semua field optional (sometimes).
- Manager hanya bisa diubah jika **belum** signed.
- Repairer yang sudah signed tidak bisa dihapus.
- Repairer yang sudah signed nama-nya tidak bisa diubah (signer immutable).
- Status `completed` → tidak bisa diubah.

---

## 7. Signature Authorization

### Manager Teknik
- Hanya user login dengan role `Manager Teknik` boleh sign.
- Match by `manager_id == signer.id` (priority) atau tolerant name match (`WorkOrderService::namesMatch`).
- Wrong user → **403**.
- Already signed → **409**.
- Signature **immutable** dan **tidak boleh diwakilkan**.

### Pelaksana (Repairer)
- Hanya user login dengan role Teknisi CNSD/TFP, Supervisor CNSD/TFP, atau Admin boleh sign.
- Resolve row by:
  1. Eksplisit `repairer_row_id` di payload (priority).
  2. Match `person_id == signer.id`.
  3. Tolerant name match.
- Wrong user → **403**.
- Already signed → **409**.
- Tidak ada di daftar pelaksana → **403** dengan pesan jelas.

### HTTP Status Codes
| Kondisi | Status |
|---|---|
| Signature tersimpan | 200 |
| Role/nama tidak cocok | 403 |
| Sudah pernah ditandatangani | 409 |
| Payload tidak valid (bukan PNG base64) | 422 |

---

## 8. Status Lifecycle

Status di-derive dari signatures + tanggal:
- `ongoing` — belum lengkap, belum melewati tanggal laporan.
- `on_hold` — belum lengkap dan tanggal sudah lewat.
- `completed` — semua signature (manager + semua repairer) sudah ada.

Status tidak diset dari frontend. Backend recalculate saat `update` dan `sign`.

---

## 9. Frontend Routes

| Path | Page |
|---|---|
| `/reporting` | `ReportingListPage` (list, filter, search) |
| `/reporting/damage-reports/new` | `ReportingDamageFormPage` (create) |
| `/reporting/damage-reports/:id` | `ReportingDamageFormPage` (edit + signature) |
| `/reporting/damage-reports/:id/print` | `ReportingDamagePrintView` |
| `/reporting/damage-reports` | redirect → `/reporting` |
| `/reports`, `/reports/create` | backward-compat redirects |

Topbar menu Reporting tampil untuk semua role authenticated kecuali yang tidak punya akses operasional (semua role kecuali user kosong).

---

## 10. Print View

- Logo AirNav asli: `/assets/icon/logoairnav.svg`
- Header tengah: judul **"LAPORAN KERUSAKAN"**
- Body: tabel 13 baris dengan kolom NO | URAIAN | DATA
- Kode hambatan: 9 kode dengan checkbox terpilih, alasan lain ditampilkan di bawah jika `AL`
- Footer signature:
  - **Kiri:** Mengetahui, Manager Teknik (1 signature box)
  - **Kanan:** Pelaksana Perbaikan dengan tabel No | Nama | Paraf
- Print **manual only** — tidak ada `setTimeout(window.print)`. Print hanya saat tombol Print PDF diklik.
- CSS print A4 portrait, hide toolbar.

---

## 11. UI Form Notes

Form dibuat modern dan mudah diisi (TIDAK kaku seperti form kertas). Print view yang harus mendekati form resmi.

Section di form:
1. Data Laporan — Tanggal, Lokasi, Fasilitas, Nama Peralatan, Modul, Kategori
2. Detail Kerusakan — Uraian + Penyebab
3. Tindakan Perbaikan — Tindakan + Dikerjakan Oleh (radio: Lokasi/Pusat)
4. Waktu Kerusakan dan Perbaikan — Mulai, Selesai, Downtime (auto-calc)
5. Kode Hambatan — dropdown 9 kode + Alasan Lain conditional
6. Penanggung Jawab — Manager Teknik dropdown + Pelaksana dynamic list

Jangan batasi pelaksana hanya satu divisi. Boleh campur Teknisi CNSD + Teknisi TFP + Supervisor CNSD + Supervisor TFP.

---

## 12. Hard Constraints

- ❌ Jangan auto-pull personel dari rostering shift untuk Reporting.
- ❌ Jangan merusak Work Order, CNSD, TFP, Grounding, Ground Check yang sudah berjalan.
- ❌ Jangan write ke `db_rostering`.
- ❌ Jangan auto-print pada page load — print hanya saat user klik tombol.
- ❌ Jangan overwrite signature yang sudah tersimpan (immutable).
- ❌ Jangan izinkan Manager Teknik atau Pelaksana yang sudah signed dihapus/diubah.
- ✅ Personel dropdown source: `local_users` dengan filter role yang spesifik.
- ✅ Format nomor LTK-YYMMDD-SEQ wajib di list, detail, dan print.
- ✅ Logo AirNav asli wajib tampil di print view.
