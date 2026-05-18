# Session Handoff — ATOMS PROJECT

> Gunakan file ini untuk orientasi di awal session baru.
> Update bagian "Current State" di akhir setiap session.

---

## Cara Pakai

**Di awal session:** Baca file ini + module `AGENTS.md` yang relevan.
**Di akhir session:** Update bagian "Current State" di bawah.

---

## Current State (per 2026-05-19 — Grounding Report module shipped)

### Perubahan Terbaru

| Fitur | Status |
|---|---|
| Backend module: GroundingReportRecord/Technician/Item + service/template/controller/requests | ✅ |
| Migrations: `grounding_report_records`, `grounding_report_technicians`, `grounding_report_items` | ✅ |
| Item template: 6 VISUAL items + 3 PENGUKURAN items | ✅ |
| Report-number format: `GROUNDING-YYMMDD-SEQ` (contoh `GROUNDING-260519-001`) | ✅ |
| Multiple records per shift allowed (berbeda peralatan) | ✅ |
| Roster integration: pull MT, Supervisor TFP, all Support technicians | ✅ |
| Signature authorization: name-match, immutable, no delegation, per-technician row | ✅ |
| Endpoints: 8 routes di bawah `/api/v1/grounding/reports` | ✅ |
| Frontend list page: API-connected, search/filter, create modal (no kantor dropdown) | ✅ |
| Frontend detail page: checklist form (VISUAL + PENGUKURAN), signature panel | ✅ |
| Frontend print view: AirNav logo, manual only, TFP-style footer signature | ✅ |
| Router: list/detail/print routes untuk Grounding | ✅ |
| Hapus GroundingCreatePage (dummy) — diganti dengan modal + detail page | ✅ |
| Hapus field "Dibuat Oleh" dan "Disetujui Oleh" — diganti signature panel | ✅ |
| Hapus dropdown "Kantor Unit Kerja" — default Cabang Surabaya | ✅ |
| Work Order, CNSD, TFP tetap berjalan | ✅ |
| Backend tests pass (2 passed) | ✅ |
| Frontend build green | ✅ |

**File diubah/ditambah session ini:**

Backend (atoms-maintenance):
- New: `app/Models/Grounding/GroundingReportRecord.php`, `GroundingReportTechnician.php`, `GroundingReportItem.php`
- New: `app/Services/Grounding/GroundingReportTemplate.php`, `GroundingReportService.php`
- New: `app/Http/Controllers/Api/V1/Grounding/GroundingReportController.php`
- New: `app/Http/Requests/Grounding/{Create,Update,Sign}GroundingReportRequest.php`
- New: 3 migrations under `database/migrations/2026_05_19_500001-500003_*`
- Modified: `routes/api.php` (Grounding route group, removed non-existent TFP TX/Tower routes)

Frontend (atoms-maintenance):
- New: `src/types/grounding.ts`
- New: `src/services/groundingReportService.ts`
- New: `src/pages/grounding/GroundingIndexPage.tsx` (rewrite — API-connected, create modal)
- New: `src/pages/grounding/GroundingReportDetailPage.tsx`
- New: `src/pages/grounding/GroundingReportPrintView.tsx`
- New: `src/pages/grounding/components/GroundingReportSignaturePanel.tsx`
- Modified: `src/router/index.tsx` — register Grounding detail/print routes, remove GroundingCreatePage

Context:
- Modified: `KIRO_TASK_CONTEXT.md` — Grounding module live
- Modified: `.agents/session-handoff.md` — current state
- New: `.agents/instructions/grounding-rules.md`
- Modified: `BACKEND_CONTEXT.md` — Grounding module section
- Modified: `.kiro/steering/index.md` — integration status

**Build/test:** `rtk php artisan migrate` ✅ 3 new migrations | `rtk php artisan route:list --path=grounding` ✅ 8 routes | `rtk test php artisan test` ✅ 2 passed | `rtk npm run build` ✅ green

### Next Steps (Prioritas)

1. **End-to-end manual test** — login via rostering, buka Grounding, klik Tambah Laporan, isi nama peralatan + lokasi, verifikasi teknisi TFP, form number GROUNDING-*, checklist VISUAL + PENGUKURAN, sign, print view.
2. **CNSD modul berikutnya** — Receiver, Glide Path, dll jika ada form referensi resmi.
3. **TFP modul berikutnya** — form TFP lain jika ada form referensi resmi.


## Previous State (per 2026-05-19 — CNSD Transmitter Meter Reading module shipped)

### Perubahan Terbaru

| Fitur | Status |
|---|---|
| Card "Transmitter" diaktifkan di CNSD index (CNSD-005) | ✅ |
| Backend module: CnsdTransmitterMeterRecord/Technician/Item + service/template/controller/requests | ✅ |
| Migrations: `cnsd_transmitter_meter_records`, `cnsd_transmitter_meter_technicians`, `cnsd_transmitter_meter_items` | ✅ |
| Partial unique index per (form_type, facility, date, shift_type) | ✅ |
| Item template: 9 groups TX Radio (Ground, ADC, CDU, APP, TMA West, TMA East, ER Makassar, ATIS, Back Up Radio) + 4 Lingkungan Kerja | ✅ |
| Form-number format: `TRANSMITTER-YYMMDD-SEQ` (contoh `TRANSMITTER-260519-001`) | ✅ |
| Form code: FORM C-1 | ✅ |
| Status dropdown: On Air/STBY (PAE primary), Online/Offline (OTE secondary, CDU) | ✅ |
| Back Up Radio status disabled/blocked (grey cells, backend rejects update) | ✅ |
| Lingkungan Kerja: 4 items (suhu, humidity, kebersihan, UPS) | ✅ |
| Roster integration: pull MT, Supervisor CNSD, all CNS technicians | ✅ |
| Signature authorization: name-match, immutable, no delegation, per-technician row | ✅ |
| Endpoints: 8 routes di bawah `/api/v1/cnsd/transmitter-meter` | ✅ |
| Frontend list page, detail/edit page (2 section tabs), signature panel, print view | ✅ |
| CnsdIndexPage: CNSD-005 aktif di CNSD_ACTIVE_ROUTES | ✅ |
| Router: list/detail/print routes untuk CNSD Transmitter Meter | ✅ |
| Print view: AirNav logo, FORM C-1, manual only (no auto-print) | ✅ |
| EQ-1, Radar, Recorder, AMSC, Work Order, TFP tetap berjalan | ✅ |
| Backend tests pass (2 passed) | ✅ |
| Frontend build green | ✅ |

**File diubah/ditambah session ini:**

Backend (atoms-maintenance):
- New: `app/Models/Cnsd/CnsdTransmitterMeterRecord.php`, `CnsdTransmitterMeterTechnician.php`, `CnsdTransmitterMeterItem.php`
- New: `app/Services/Cnsd/CnsdTransmitterMeterTemplate.php`, `CnsdTransmitterMeterService.php`
- New: `app/Http/Controllers/Api/V1/Cnsd/CnsdTransmitterMeterController.php`
- New: `app/Http/Requests/Cnsd/{Create,Update,Sign}CnsdTransmitterMeterRequest.php`
- New: `app/Exceptions/CnsdTransmitterMeterDuplicateException.php`
- New: 3 migrations under `database/migrations/2026_05_19_400001-400003_*`
- Modified: `routes/api.php` (Transmitter route group)

Frontend (atoms-maintenance):
- New: `src/types/cnsdTransmitter.ts`
- New: `src/services/cnsdTransmitterMeterService.ts`
- New: `src/pages/cnsd/CnsdTransmitterMeterListPage.tsx`
- New: `src/pages/cnsd/CnsdTransmitterMeterDetailPage.tsx`
- New: `src/pages/cnsd/CnsdTransmitterMeterPrintView.tsx`
- New: `src/pages/cnsd/components/CnsdTransmitterMeterSignaturePanel.tsx`
- Modified: `src/pages/cnsd/CnsdIndexPage.tsx` — CNSD-005 aktif di CNSD_ACTIVE_ROUTES
- Modified: `src/router/index.tsx` — register Transmitter list/detail/print routes
- Modified: `src/data/mockData.ts` — CNSD-005 `is_active_mvp: true`

Context:
- Modified: `KIRO_TASK_CONTEXT.md` — CNSD-005 Transmitter live, format nomor Transmitter
- Modified: `.agents/session-handoff.md` — current state

**Build/test:** `rtk php artisan migrate` ✅ 3 new migrations | `rtk php artisan route:list --path=cnsd/transmitter` ✅ 8 routes | `rtk test php artisan test` ✅ 2 passed | `rtk npm run build` ✅ green

### Next Steps (Prioritas)

1. **End-to-end manual test** — login via rostering, buka CNSD, klik card Transmitter, buat record, verifikasi teknisi CNSD, form number TRANSMITTER-*, 2 section tabs, print view.
2. **CNSD modul berikutnya** — Receiver, Glide Path, dll jika ada form referensi resmi.
3. **Notification adoption** untuk Transmitter (pola EQ-1 sudah ada, tinggal adopt).


## Previous State (per 2026-05-19 — Manual Form Development Hook dibuat)

### Perubahan Terbaru

| Fitur | Status |
|---|---|
| `.kiro/hooks/manual-form-workflow.md` dibuat | ✅ |
| `.kiro/steering/index.md` diupdate — referensi hook + section Form Development Hooks | ✅ |
| `KIRO_TASK_CONTEXT.md` diupdate — section Manual Form Development Hook | ✅ |
| `.agents/agents.md` diupdate — tambah Manual Form Hook ke Key Instructions | ✅ |

**File dibuat/diubah session ini:**

Context:
- New: `.kiro/hooks/manual-form-workflow.md` — hook workflow 16 section
- Modified: `.kiro/steering/index.md` — referensi hook, section Form Development Hooks
- Modified: `KIRO_TASK_CONTEXT.md` — section 10 Manual Form Development Hook
- Modified: `.agents/agents.md` — tambah Manual Form Hook ke Key Instructions table

**Tidak ada perubahan file aplikasi (backend/frontend).**

### Ringkasan Hook

`.kiro/hooks/manual-form-workflow.md` mencakup 16 section:
1. Tujuan Hook
2. File Wajib Dibaca
3. Pull/Sync Wajib
4. Hard Constraints
5. Pola Backend Form Manual
6. Pola Frontend Form Manual
7. Pola Roster (CNSD vs TFP)
8. Pola Signature + Footer Print
9. Pola Nomor Form
10. Pola Template Item
11. Pola UI Detail/Edit
12. Pola Print View (termasuk larangan auto-print)
13. Testing Wajib (command + manual checklist)
14. Dokumentasi Wajib
15. Commit dan Push Wajib
16. Output Akhir Standar

### Next Steps (Prioritas)

1. **CNSD modul berikutnya** — Transmitter, Receiver, dll jika ada form referensi resmi.
2. **TFP modul berikutnya** — form TFP lain jika ada form referensi resmi.
3. **Notification adoption** untuk AMSC, Radar, Recorder (pola EQ-1 sudah ada).


## Previous State (per 2026-05-19 — CNSD AMSC print footer fix)

### Perubahan Terbaru

| Fitur | Status |
|---|---|
| Migration: tambah `day_name` + `time_filled` ke `cnsd_amsc_meter_records` | ✅ |
| Model: `day_name` + `time_filled` masuk `$fillable` | ✅ |
| Service: set `day_name` + `time_filled` saat create, refresh `time_filled` saat update | ✅ |
| Controller: expose `day_name` + `time_filled` di summary + detail response | ✅ |
| Frontend type: tambah `day_name` + `time_filled` ke `CnsdAmscMeterRecordDetail` + Summary | ✅ |
| Print view: footer sekarang menampilkan Hari/Tanggal/Jam dari record | ✅ |
| Print view: signature footer mengikuti pola EQ-1/Recorder (border-black, dashed box, signed_at) | ✅ |
| Print view: "Tidak ada supervisor pada shift ini" jika supervisor null | ✅ |
| Print view: "Manager Teknik tidak ditugaskan" jika manager null | ✅ |
| Print view: tetap manual only (no auto-print) | ✅ |
| Backend tests pass (2 passed) | ✅ |
| Frontend build green | ✅ |

**File diubah session ini:**

Backend:
- Modified: `app/Models/Cnsd/CnsdAmscMeterRecord.php` — tambah `day_name`, `time_filled` ke `$fillable`
- Modified: `app/Services/Cnsd/CnsdAmscMeterService.php` — set `day_name`/`time_filled` saat create, refresh `time_filled` saat update
- Modified: `app/Http/Controllers/Api/V1/Cnsd/CnsdAmscMeterController.php` — expose `day_name`/`time_filled` di response
- New: `database/migrations/2026_05_19_300004_add_time_fields_to_cnsd_amsc_meter_records.php`

Frontend:
- Modified: `src/types/cnsdAmsc.ts` — tambah `day_name`, `time_filled` ke interfaces
- Modified: `src/pages/cnsd/CnsdAmscMeterPrintView.tsx` — rewrite footer dengan waktu pelaksanaan + signature pola EQ-1/Recorder

**Build/test:** `rtk php artisan migrate` ✅ 1 new migration | `rtk test php artisan test` ✅ 2 passed | `rtk npm run build` ✅ green

### Next Steps (Prioritas)

1. **End-to-end manual test** — buka AMSC print view, verifikasi Hari/Tanggal/Jam tampil, signature footer mirip EQ-1.
2. **CNSD modul berikutnya** — Transmitter, Receiver, dll jika ada form referensi resmi.
3. **Notification adoption** untuk AMSC (pola EQ-1 sudah ada, tinggal adopt).


## Previous State (per 2026-05-19 — CNSD AMSC Meter Reading module shipped)

### Perubahan Terbaru

| Fitur | Status |
|---|---|
| Card "AMSC" diaktifkan di CNSD index (CNSD-004) | ✅ |
| Backend module: CnsdAmscMeterRecord/Technician/Item + service/template/controller/requests | ✅ |
| Migrations: `cnsd_amsc_meter_records`, `cnsd_amsc_meter_technicians`, `cnsd_amsc_meter_items` | ✅ |
| Partial unique index per (form_type, facility, date, shift_type) | ✅ |
| Item template: 45 items (4 sections: Front Panel 5, PSU 5, Channel 32, Lingkungan 3) | ✅ |
| Form-number format: `AMSC-YYMMDD-SEQ` (contoh `AMSC-260519-001`) | ✅ |
| Channel AMSC: 32 channels with default address/status/keterangan from paper form | ✅ |
| Roster integration: pull MT, Supervisor CNSD, all CNS technicians | ✅ |
| Signature authorization: name-match, immutable, no delegation, per-technician row | ✅ |
| Endpoints: 8 routes di bawah `/api/v1/cnsd/amsc-meter` | ✅ |
| Frontend list page, detail/edit page (4 section tabs), signature panel, print view | ✅ |
| CnsdIndexPage: CNSD-004 aktif di CNSD_ACTIVE_ROUTES | ✅ |
| Router: list/detail/print routes untuk CNSD AMSC Meter | ✅ |
| Dropdown: Normal/Alrm, √/-, OK/Not (Front Panel), Normal/U-S/Fault (Channel status) | ✅ |
| Print view: CNSD-style signature footer, logo AirNav asli, no auto-print | ✅ |
| EQ-1, Radar, Recorder, Work Order, TFP tetap berjalan | ✅ |
| Backend tests pass (2 passed) | ✅ |
| Frontend build green | ✅ |

**File diubah/ditambah session ini:**

Backend (atoms-maintenance):
- New: `app/Models/Cnsd/CnsdAmscMeterRecord.php`, `CnsdAmscMeterTechnician.php`, `CnsdAmscMeterItem.php`
- New: `app/Services/Cnsd/CnsdAmscMeterTemplate.php`, `CnsdAmscMeterService.php`
- New: `app/Http/Controllers/Api/V1/Cnsd/CnsdAmscMeterController.php`
- New: `app/Http/Requests/Cnsd/{Create,Update,Sign}CnsdAmscMeterRequest.php`
- New: `app/Exceptions/CnsdAmscMeterDuplicateException.php`
- New: 3 migrations under `database/migrations/2026_05_19_300001-300003_*`
- Modified: `routes/api.php` (AMSC route group)
- Modified: `BACKEND_CONTEXT.md` (AMSC module section)

Frontend (atoms-maintenance):
- New: `src/types/cnsdAmsc.ts`
- New: `src/services/cnsdAmscMeterService.ts`
- New: `src/pages/cnsd/CnsdAmscMeterListPage.tsx`
- New: `src/pages/cnsd/CnsdAmscMeterDetailPage.tsx`
- New: `src/pages/cnsd/CnsdAmscMeterPrintView.tsx`
- New: `src/pages/cnsd/components/CnsdAmscMeterSignaturePanel.tsx`
- Modified: `src/pages/cnsd/CnsdIndexPage.tsx` — CNSD-004 aktif di CNSD_ACTIVE_ROUTES
- Modified: `src/router/index.tsx` — register AMSC list/detail/print routes
- Modified: `src/data/mockData.ts` — CNSD-004 `is_active_mvp: true`

Context:
- Modified: `KIRO_TASK_CONTEXT.md` — CNSD-004 AMSC live, format nomor AMSC

**Build/test:** `rtk php artisan migrate` ✅ 3 new migrations | `rtk php artisan route:list --path=cnsd/amsc` ✅ 8 routes | `rtk test php artisan test` ✅ 2 passed | `rtk npm run build` ✅ green

### Next Steps (Prioritas)

1. **End-to-end manual test** — login via rostering, buka CNSD, klik card AMSC, buat record, verifikasi teknisi CNSD, form number AMSC-*, 4 section tabs, print view.
2. **CNSD modul berikutnya** — Transmitter, Receiver, dll jika ada form referensi resmi.
3. **Notification adoption** untuk AMSC (pola EQ-1 sudah ada, tinggal adopt).


## Previous State (per 2026-05-19 — TFP AOB Lantai 1 & 2 module shipped)

### Perubahan Terbaru

| Fitur | Status |
|---|---|
| Card "AOB Lantai 1 & 2" diaktifkan di TFP index (TFP-002) | ✅ |
| Backend module: TfpAobLt12Record/Technician/Item/Facility + service/template/controller/requests | ✅ |
| Migrations: `tfp_aob_lt12_records`, `tfp_aob_lt12_technicians`, `tfp_aob_lt12_items`, `tfp_aob_lt12_facilities` | ✅ |
| Partial unique index per (form_type, date, shift_type) | ✅ |
| Item template: 21 parameters, 24 fasilitas | ✅ |
| Form-number format: `TFP-AOBLT12-YYMMDD-SEQ` (contoh `TFP-AOBLT12-260519-001`) | ✅ |
| 6 panel columns single-value (no Input/Output split): A05, A06, A07, A08, A22, A09 | ✅ |
| Roster integration: pull MT, Supervisor TFP (Support grade≥13), all Support technicians | ✅ |
| Signature authorization: name-match, immutable, no delegation, per-technician row | ✅ |
| Endpoints: 8 routes di bawah `/api/v1/tfp/aob-lt12` | ✅ |
| Frontend list page, detail/edit page, signature panel, print view | ✅ |
| TfpIndexPage: TFP-002 aktif di TFP_ACTIVE_ROUTES | ✅ |
| Router: list/detail/print routes untuk TFP AOB Lt12 | ✅ |
| Disabled cell rendering (grey cells sesuai form resmi) | ✅ |
| Dropdown Mode (Auto/Manual), Suplai Aktif (PLN/UPS), Kondisi (Baik/Normal/Tidak Baik) | ✅ |
| Print view: CNSD-style signature footer, logo AirNav asli, no auto-print | ✅ |
| AOB Ground, CNSD, Work Order tetap berjalan | ✅ |
| Backend tests pass (2 passed) | ✅ |
| Frontend build green | ✅ |

**File diubah/ditambah session ini:**

Backend (atoms-maintenance):
- New: `app/Models/Tfp/TfpAobLt12Record.php`, `TfpAobLt12Technician.php`, `TfpAobLt12Item.php`, `TfpAobLt12Facility.php`
- New: `app/Services/Tfp/TfpAobLt12Template.php`, `TfpAobLt12Service.php`
- New: `app/Http/Controllers/Api/V1/Tfp/TfpAobLt12Controller.php`
- New: `app/Http/Requests/Tfp/{Create,Update,Sign}TfpAobLt12Request.php`
- New: `app/Exceptions/TfpAobLt12DuplicateException.php`
- New: 4 migrations under `database/migrations/2026_05_19_200001-200004_*`
- Modified: `routes/api.php` (TFP AOB Lt12 route group)

Frontend (atoms-maintenance):
- New: `src/types/tfpAobLt12.ts`
- New: `src/services/tfpAobLt12Service.ts`
- New: `src/pages/tfp/TfpAobLt12ListPage.tsx`
- New: `src/pages/tfp/TfpAobLt12DetailPage.tsx`
- New: `src/pages/tfp/TfpAobLt12PrintView.tsx`
- New: `src/pages/tfp/components/TfpAobLt12SignaturePanel.tsx`
- Modified: `src/pages/tfp/TfpIndexPage.tsx` — TFP-002 aktif di TFP_ACTIVE_ROUTES
- Modified: `src/router/index.tsx` — register Lt12 list/detail/print routes
- Modified: `src/data/mockData.ts` — TFP-002 `is_active_mvp: true`

**Build/test:** `rtk php artisan migrate` ✅ 4 new migrations | `rtk php artisan route:list --path=tfp` ✅ 16 routes (8+8) | `rtk test php artisan test` ✅ 2 passed | `rtk npm run build` ✅ green

### Next Steps (Prioritas)

1. **End-to-end manual test** — login via rostering, buka TFP, klik card AOB Lantai 1 & 2, buat record, verifikasi teknisi TFP, form number TFP-AOBLT12-*, disabled cells, print view.
2. **TFP modul berikutnya** — AOB Lantai lain jika ada form referensi resmi.
3. **Notification adoption** untuk TFP (pola EQ-1 sudah ada, tinggal adopt).


## Previous State (per 2026-05-19 — TFP AOB Ground layout restore + timezone fix)

### Perubahan Terbaru (Bugfix Session #3)

| Issue | Root Cause | Fix | Status |
|---|---|---|---|
| UI detail form berantakan (unified table terlalu padat, warna terlalu gelap) | Session sebelumnya menggabungkan parameter + fasilitas ke satu `<table>` 13 kolom dengan header `bg-slate-800` — terlalu padat dan tidak user-friendly | Restore ke dua panel terpisah: kiri = "Parameter Pengukuran" card, kanan = "Kondisi Fasilitas" card. Header tabel pakai `bg-sky-800` (lebih lembut). Hapus teks "Mengikuti format form resmi" | ✅ Fixed |
| Jam masih menampilkan `18:38` (UTC, bukan WIB) | Backend `config/app.php` timezone = `'UTC'`. `now()->format('H:i')` menghasilkan UTC time, 7 jam lebih awal dari WIB | Ubah `'timezone' => 'Asia/Jakarta'` di `config/app.php` (sama dengan atoms-rostering) | ✅ Fixed |
| Merge cell baris 12-21 belum sesuai form asli | Rows 18-19 (Mode/Suplai) dan 20-21 (KWH/Suhu) masih pakai 8 sel terpisah | Rows 18-19: render 6 `<td>` dengan `colSpan=2` per panel group (COS span2 dropdown, ATS span2 dropdown, UPS A span2 grey, UPS B span2 grey). Rows 20-21: render `<td colSpan={8}>` single input | ✅ Fixed |
| Print signature belum mirip CNSD | Sudah diperbaiki di session sebelumnya | Verified: flex 3 kolom (teknisi tabel + supervisor + manager), "Belum TTD" italic, dashed-border kotak kosong | ✅ Verified |
| Print auto-print | Sudah diperbaiki di session sebelumnya | Verified: tidak ada `setTimeout(window.print)`, hanya button onClick | ✅ Verified |

### File diubah session ini

Frontend (atoms-maintenance):
- Modified: `src/pages/tfp/TfpAobGroundDetailPage.tsx`
  - Restore ke dua panel terpisah (`grid xl:grid-cols-[1fr_380px]`)
  - Header tabel: `bg-sky-800` (row 1) + `bg-sky-700` (row 2 Input/Output)
  - Rows 18-19 (Mode/Suplai): inline JSX dengan `colSpan=2` per panel group
  - Rows 20-21 (KWH/Suhu): `<td colSpan={8}>` single input
  - CellInput disederhanakan: disabled → `bg-slate-200 rounded`, completed → `<span>`, default → `<input>`
  - Hapus teks "Mengikuti format form resmi"

Backend (atoms-maintenance):
- Modified: `config/app.php` — `'timezone' => 'Asia/Jakarta'` (dari `'UTC'`)

Documentation:
- Modified: `.agents/session-handoff.md` (section ini)
- Modified: `.agents/instructions/tfp-rules.md` (timezone + layout rules)
- Modified: `BACKEND_CONTEXT.md` (timezone note)

**Build/test:** `rtk php artisan route:list --path=tfp` ✅ 8 routes | `rtk test php artisan test` ✅ 2 passed | `rtk npm run build` ✅ green

### Akar Masalah

**1. UI detail form berantakan**

Session sebelumnya menggabungkan parameter + fasilitas ke satu `<table>` 13 kolom dengan header `bg-slate-800` (sangat gelap) dan `bg-sky-700` (kontras tinggi). Hasilnya terlalu padat, font tidak konsisten, dan ada teks "Mengikuti format form resmi" yang tidak diperlukan. User lebih suka dua panel terpisah yang clean.

Fix: restore ke dua panel terpisah. Header tabel pakai `bg-sky-800` (lebih lembut dari `bg-slate-800`). Hapus teks tidak perlu.

**2. Jam menampilkan UTC bukan WIB**

`config/app.php` di atoms-maintenance menggunakan `'timezone' => 'UTC'` (default Laravel). `now()->format('H:i')` menghasilkan UTC time. Jika user menyimpan pukul 20:15 WIB, backend menyimpan `13:15` (UTC-7). Atoms-rostering sudah menggunakan `'Asia/Jakarta'`.

Fix: ubah ke `'Asia/Jakarta'`. Sekarang `now()->format('H:i')` menghasilkan WIB time yang benar.

**3. Merge cell baris 12-21**

Rows 18-19 (Mode/Suplai Aktif) sebelumnya menggunakan 8 sel terpisah dengan CellInput yang mendeteksi row type dan merender dropdown atau div kosong. Hasilnya: 2 sel terpisah (Input + Output) per panel, bukan 1 sel merged seperti form asli.

Fix: render rows 18-19 dengan inline JSX menggunakan `colSpan=2` per panel group. Rows 20-21 menggunakan `colSpan=8` untuk single input yang spans semua panel columns.

### Next Steps (Prioritas)

1. **Manual test** — user buka detail TFP, simpan, verifikasi jam berubah ke WIB now. Verifikasi rows 18-21 merge cell. Verifikasi print signature 3 kolom CNSD-style.
2. **Commit + push** sudah dilakukan di session ini.
3. Lanjut TFP modul berikutnya atau Notification adoption.


## Previous State (per 2026-05-19 — TFP AOB Ground time-filled, unified table, CNSD-style signature)

### Perubahan Terbaru (Bugfix Session #2)

| Issue | Root Cause | Fix | Status |
|---|---|---|---|
| `time_filled` (jam) tidak update saat user simpan perubahan | Backend hanya set `time_filled = now()->format('H:i')` di `createRecord()`. `updateItems()` dan `updateFacilities()` tidak menyentuh kolom ini. | Tambah `$record->time_filled = now()->format('H:i'); $record->save();` di akhir `updateItems()` dan `updateFacilities()` (sebelum return fresh). Setiap Simpan Perubahan = snapshot waktu baru | ✅ Fixed |
| Detail/edit form: tabel parameter & fasilitas TIDAK menyatu (dua tabel terpisah) | Detail page memakai `<div className="grid xl:grid-cols-[1fr_380px]">` — flex container dengan 2 `<table>` berbeda; tidak ada cell yang merge antar kolom | Refactor ke SATU `<table>` 13 kolom (No, Parameter, COS×2, ATS×2, UPS A×2, UPS B×2, Nama Fasilitas, Kondisi, Keterangan). Header pakai colSpan/rowSpan. Index-aligned: parameter row N sejajar facility row N | ✅ Fixed |
| Print signature footer berbeda dari CNSD | Print view TFP memakai 1 `<table>` dengan 5 kolom (No/Nama/Paraf/Supervisor/Manager) plus rowSpan supervisor & manager — beda layout dari CNSD | Re-implement persis pola `CnsdReadinessPrintView`: section terpisah dengan flex 3 kolom: kiri = tabel teknisi (No/Nama/Paraf), tengah = supervisor (signature/dashed-empty/italic note jika tidak ada), kanan = manager teknik (signature/dashed-empty) | ✅ Fixed |
| Detail page disabled cells tidak terlihat menyatu seperti form resmi | `<td>` tidak punya `bg-slate-300` untuk dynamic-disabled cells (Mode/Suplai/SingleValue) — hanya untuk hard-disabled via `is_disabled_map` | `<td>` sekarang dapat class `bg-slate-300` jika cell hard-disabled OR dynamic-disabled (Mode/Suplai/SingleValue rules). CellInput inner div jadi transparan (h-8) untuk konsistensi tinggi row | ✅ Fixed |
| Print masih tidak auto-print (verified) | — | Tidak ada `setTimeout(window.print)`, tidak ada auto-print useEffect. Hanya button onClick. | ✅ Verified |

### File diubah session ini

Frontend (atoms-maintenance):
- Modified: `src/pages/tfp/TfpAobGroundDetailPage.tsx`
  - Hapus konstanta `PANEL_GROUPS` (tidak terpakai setelah refactor)
  - Refactor body dari "Two-panel grid" → SATU `<table>` 13 kolom dengan `colSpan`/`rowSpan` di header
  - `<td>` panel cells dapat `bg-slate-300` kelas saat hard-disabled atau dynamic-disabled (Mode/Suplai/SingleValue) — agar terlihat blok grey menyatu
  - CellInput inner div untuk disabled cell dijadikan transparan, parent `<td>` provide bg
- Modified: `src/pages/tfp/TfpAobGroundPrintView.tsx`
  - Hapus `minTechRows`/`techRowCount` (unused)
  - Refactor signature footer dari single 5-column `<table>` → flex 3 kolom mirroring `CnsdReadinessPrintView`:
    - Kiri: tabel teknisi (No/Nama/Paraf) dengan "Belum TTD" italic placeholder
    - Tengah: supervisor (signature image atau dashed-border kotak kosong, atau "Tidak ada supervisor pada shift ini" italic)
    - Kanan: manager teknik (signature image atau dashed-border atau "Manager Teknik tidak ditugaskan" italic)

Backend (atoms-maintenance):
- Modified: `app/Services/Tfp/TfpAobGroundService.php`
  - `updateItems()` — refresh `$record->time_filled = now()->format('H:i')` setelah loop fill items
  - `updateFacilities()` — refresh `$record->time_filled` setelah loop facilities. Setiap Simpan Perubahan = waktu baru di backend.

Documentation:
- Modified: `.agents/session-handoff.md` (section ini)
- Modified: `.agents/instructions/tfp-rules.md` (time_filled rule + signature print pattern)
- Modified: `BACKEND_CONTEXT.md` (time_filled refresh on update)

**Build/test:** `rtk php artisan route:list --path=tfp` ✅ 8 routes intact | `rtk test php artisan test` ✅ 2 passed | `rtk npm run build` ✅ green | TypeScript clean

### Akar Masalah

**1. Jam tidak update saat simpan**

Field `time_filled` di-set sekali saat record pertama kali dibuat (`createRecord`) sesuai pola CNSD. Tidak ada flow yang memperbarui kolom ini. Akibatnya, jam yang tampil di header detail/edit dan print view selalu menunjukkan waktu pembuatan record (mis. `18:38`), bukan waktu user mengisi.

User mengharapkan: form fisik mencatat "jam saat petugas mengambil reading" — sehingga setiap kali Simpan Perubahan, jam harus update ke `now()`.

Fix: backend menulis `$record->time_filled = now()->format('H:i')` di akhir `updateItems()` dan `updateFacilities()`. Tanggal dan hari tetap dari `date` (tidak diubah) sehingga konsisten dengan record header.

**2. Tabel parameter & fasilitas tidak menyatu**

Detail page memakai 2 tabel terpisah dalam flex container `xl:grid-cols-[1fr_380px]`:
- Tabel parameter (kiri): `<table>` 10 kolom
- Fasilitas (kanan): bukan tabel, melainkan div grid `[1fr_90px_1fr]`

Ini tidak bisa merge cell antar tabel dan tidak align secara baris. Visual akhirnya tampak seperti dua card terpisah, bukan dokumen unified.

Fix: refactor ke SATU `<table>` 13 kolom: `[No, Parameter, COS Input, COS Output, ATS Input, ATS Output, UPS A Input, UPS A Output, UPS B Input, UPS B Output, Nama Fasilitas, Kondisi, Keterangan]`. Header pakai `colSpan=2` (Panel COS/ATS/UPS A/UPS B) dan `rowSpan=2` (No, Parameter, Nama Fasilitas, Kondisi, Keterangan). Body di-render row-per-index — parameter `record.items[idx]` sejajar dengan facility `record.facilities[idx]`.

**3. Signature print belum mirip CNSD**

Print view TFP memakai 1 `<table>` dengan 5 kolom dan rowSpan untuk supervisor/manager. CNSD pakai pendekatan berbeda: section terpisah dari tabel utama, flex 3 kolom, dengan dashed-border kotak kosong saat belum TTD dan italic placeholder text saat assignment kosong.

Fix: copy pola persis dari `CnsdReadinessPrintView.tsx`:
- Kiri (flex-1): tabel teknisi 3 kolom (No/Nama/Paraf) dengan border CNSD style + "Belum TTD" italic placeholder
- Tengah (w-24%): supervisor — signature image, dashed border kotak kosong, atau "Tidak ada supervisor pada shift ini" italic
- Kanan (w-24%): manager teknik — signature image, dashed border kotak kosong, atau "Manager Teknik tidak ditugaskan" italic

### Next Steps (Prioritas)

1. **Manual test** — user buka detail TFP, ubah salah satu cell, klik Simpan, verifikasi jam berubah ke now. Klik Print, verifikasi tabel besar menyatu, signature 3 kolom mirip CNSD.
2. **Commit + push** session bugfix #2.
3. Lanjut TFP modul berikutnya atau Notification adoption (pola EQ-1).


## Previous State (per 2026-05-19 — TFP AOB Ground print + form layout refinement)

### Perubahan Terbaru (Bugfix Session)

| Issue | Root Cause | Fix | Status |
|---|---|---|---|
| Print view auto-prints 2 kali saat halaman dibuka | `setTimeout(() => window.print(), 500)` di `useEffect` + React StrictMode → effect dijalankan 2× di dev mode | Hapus auto-print. `window.print()` hanya dipanggil saat klik tombol Print PDF (sama dengan CNSD pattern) | ✅ Fixed |
| Print view pakai placeholder logo (lingkaran biru) bukan logo asli AirNav | TFP print view tidak reference asset path | Reuse `/assets/icon/logoairnav.svg` (pattern sama dengan CNSD/Work Order print) | ✅ Fixed |
| Layout print pakai grid div terpisah, tidak menyerupai dokumen | Tabel parameter dan fasilitas dibuat sebagai 2 tabel terpisah dalam flex container | Refactor ke single `<table>` HTML besar dengan colSpan/rowSpan untuk kop, sub-header (Input/Output), dan area fasilitas (Nama Fasilitas/Kondisi/Keterangan sebagai kolom rightmost) | ✅ Fixed |
| Disabled cells terlihat sebagai input kecil terpisah, bukan blok grey menyatu | Detail page `bg-slate-200` terlalu light + tidak ada visual cue blok merge | Detail: ubah ke `bg-slate-300/80` agar lebih kontras dan mirip form resmi. Print: `bg-gray-300` tanpa border-radius — render sebagai blok grey solid | ✅ Fixed |
| Print kecil/sulit dibaca | Kolom width terlalu kecil + font 8px | Re-ukur kolom (30-40px per panel cell, 110px parameter, 120px facility), font 9px untuk body 10px untuk header, A4 landscape, max-width 290mm | ✅ Improved |
| Backend bisa terima patch ke disabled cells | `updateItems()` tidak strip disabled columns | Tambah guard: baca `is_disabled_map` per item, strip disabled column keys dari payload sebelum `fill()`. is_disabled_map = source of truth | ✅ Fixed |

### File diubah session ini

Frontend (atoms-maintenance):
- Modified: `src/pages/tfp/TfpAobGroundPrintView.tsx` — full rewrite. Single big HTML table dengan colSpan/rowSpan, logo asli AirNav, header tengah judul, no auto-print, A4 landscape. Cleanup function untuk cancellation di useEffect.
- Modified: `src/pages/tfp/TfpAobGroundDetailPage.tsx` — disabled cell visual update (`bg-slate-300/80`) untuk merge appearance lebih jelas.

Backend (atoms-maintenance):
- Modified: `app/Services/Tfp/TfpAobGroundService.php` — `updateItems()` strip disabled columns berdasarkan `is_disabled_map` per row sebelum fill.

Documentation:
- Modified: `.agents/session-handoff.md` (section ini)
- Modified: `.agents/instructions/tfp-rules.md` (print + disabled cell rules)

**Build/test:** `rtk php artisan route:list --path=tfp` ✅ 8 routes | `rtk test php artisan test` ✅ 2 passed | `rtk npm run build` ✅ green

### Akar Masalah (Root Cause Analysis)

**Print spam 2 kali**

`useEffect` di TFP print view memanggil `setTimeout(() => window.print(), 500)` setiap data record sukses dimuat. Di React StrictMode (dev mode), `useEffect` dijalankan dua kali untuk men-detect side-effect bug — masing-masing menjadwalkan `window.print()` lewat setTimeout. Hasilnya: 2 dialog print muncul.

Pola CNSD print view (Readiness, Radar, Recorder) tidak pernah memanggil `window.print()` di dalam useEffect — hanya pada `onClick` tombol toolbar. Solusinya: hapus auto-print, ikuti pola CNSD.

**Layout merge cell**

Form resmi terdiri dari satu tabel besar dimana kolom kiri (parameter dengan 4 panel × 2 sub-kolom Input/Output) berbagi baris yang sama dengan kolom kanan (Nama Fasilitas/Kondisi/Keterangan). Implementasi sebelumnya memakai dua `<table>` terpisah dalam flex container — tidak bisa merge cell antar tabel, dan baris pada kedua tabel tidak align secara visual.

Solusi: render sebagai SATU `<table>` 13 kolom (No, Parameter, COS×2, ATS×2, UPS A×2, UPS B×2, Nama Fasilitas, Kondisi, Keterangan) dengan baris yang index-aligned. Header pakai `colSpan=2` (Panel COS, ATS, UPS A, UPS B) dan `rowSpan=2` (No, Parameter, Nama Fasilitas, Kondisi, Keterangan).

**Disabled cell appearance**

Form resmi menampilkan area abu-abu sebagai blok solid yang tampak menyatu (merge visual) untuk kolom yang tidak relevan dengan parameter (mis. baris 14-17 Battery hanya untuk UPS, Panel COS/ATS jadi grey). Detail page sebelumnya pakai `bg-slate-200` (terlalu light) dengan border-radius — terlihat seperti input kecil terpisah.

Solusi: 
- Detail: `bg-slate-300/80` (lebih kontras, mendekati abu-abu form)
- Print: `bg-gray-300` tanpa rounded — tampak sebagai blok solid yang merge dengan border tabel
- Backend: enforce dengan `is_disabled_map` guard di `updateItems()` agar bahkan jika frontend menulis ke disabled cell, backend strip dari payload

### Next Steps (Prioritas)

1. **Manual test print** — buka detail TFP → klik Print → verifikasi:
   - Dialog print TIDAK muncul otomatis saat halaman dibuka
   - Klik Print PDF → dialog muncul SEKALI
   - Logo AirNav tampil
   - Tabel besar menyatu (parameter + fasilitas)
   - Disabled cells abu-abu blok solid
   - Footer signature lengkap
2. **Commit + push** session bugfix ini.
3. Lanjut TFP modul berikutnya atau modul CNSD yang masih Coming Soon.


## Previous State (per 2026-05-19 — TFP Performance Check AOB Lantai Ground module shipped)

### Perubahan Terbaru

| Fitur | Status |
|---|---|
| Card "Performance Check AOB Lantai Ground" diaktifkan di TFP index (sebelumnya dummy form) | ✅ |
| Backend module: TfpAobGroundRecord/Technician/Item/Facility + service/template/controller/requests | ✅ |
| Migrations: `tfp_aob_ground_records`, `tfp_aob_ground_technicians`, `tfp_aob_ground_items`, `tfp_aob_ground_facilities` | ✅ |
| Partial unique index per (form_type, date, shift_type) | ✅ |
| Item template: 21 parameters (row Suhu Ruang ARO DIHAPUS), 17 fasilitas | ✅ |
| Form-number format: `TFP-AOBLTGND-YYMMDD-SEQ` (contoh `TFP-AOBLTGND-260519-001`) | ✅ |
| Roster integration: pull MT, Supervisor TFP (Support grade≥13), all Support technicians | ✅ |
| Signature authorization: name-match, immutable, no delegation, per-technician row | ✅ |
| Endpoints: 8 routes di bawah `/api/v1/tfp/aob-ground` | ✅ |
| Frontend list page, detail/edit page, signature panel, print view | ✅ |
| TfpIndexPage diupdate ke CNSD-style active routes map (TFP_ACTIVE_ROUTES) | ✅ |
| Router diupdate: list/detail/print routes untuk TFP AOB Ground | ✅ |
| Disabled cell rendering (grey cells sesuai form resmi) | ✅ |
| Dropdown Mode (Auto/Manual), Suplai Aktif (PLN/UPS, PLN 1/PLN 2), Kondisi (Baik/Normal/Tidak Baik) | ✅ |
| CNSD, Work Order tetap berjalan (tidak ada refactor besar yang merusak) | ✅ |
| Backend tests pass (2 passed) | ✅ |
| Frontend build green | ✅ |

**File diubah/ditambah session ini:**

Backend (atoms-maintenance):
- New: `app/Models/Tfp/TfpAobGroundRecord.php`, `TfpAobGroundTechnician.php`, `TfpAobGroundItem.php`, `TfpAobGroundFacility.php`
- New: `app/Services/Tfp/TfpAobGroundTemplate.php`, `TfpAobGroundService.php`
- New: `app/Http/Controllers/Api/V1/Tfp/TfpAobGroundController.php`
- New: `app/Http/Requests/Tfp/{Create,Update,Sign}TfpAobGroundRequest.php`
- New: `app/Exceptions/TfpAobGroundDuplicateException.php`
- New: 4 migrations under `database/migrations/2026_05_19_100001-100004_*`
- Modified: `routes/api.php` (TFP route group)
- Modified: `BACKEND_CONTEXT.md` (TFP module section)

Frontend (atoms-maintenance):
- New: `src/types/tfpAobGround.ts`
- New: `src/services/tfpAobGroundService.ts`
- New: `src/pages/tfp/TfpAobGroundListPage.tsx`
- New: `src/pages/tfp/TfpAobGroundDetailPage.tsx`
- New: `src/pages/tfp/TfpAobGroundPrintView.tsx`
- New: `src/pages/tfp/components/TfpAobGroundSignaturePanel.tsx`
- Modified: `src/pages/tfp/TfpIndexPage.tsx` — TFP_ACTIVE_ROUTES map, TFP-001 aktif
- Modified: `src/router/index.tsx` — register TFP list/detail/print routes, remove old dummy form route

Documentation:
- Modified: `BACKEND_CONTEXT.md` — TFP module section
- Modified: `.agents/session-handoff.md` — current state
- Created: `.agents/instructions/tfp-rules.md` — TFP module rules
- Modified: `KIRO_TASK_CONTEXT.md` — TFP current state section

**Build/test:** `rtk php artisan migrate` ✅ 4 new migrations | `rtk php artisan route:list --path=tfp` ✅ 8 routes | `rtk test php artisan test` ✅ 2 passed | `rtk npm run build` ✅ green

### Next Steps (Prioritas)

1. **End-to-end SSO walkthrough** untuk TFP AOB Ground — perlu 4 server berjalan + akun SSO rostering.
   - Pastikan teknisi yang masuk adalah `employee_type='Support'` (bukan CNS).
   - Pastikan row Suhu Ruang ARO tidak ada.
   - Pastikan disabled cells grey dan tidak bisa diisi.
   - Pastikan dropdown Mode, Suplai Aktif, Kondisi berfungsi.
2. **Commit + push** session ini.
3. **TFP modul berikutnya** — AOB Lantai 1 & 2, atau modul TFP lain jika ada form referensi resmi.
4. **CNSD modul berikutnya** — AMSC, Transmitter, dll jika ada form referensi resmi.
5. Notification adoption untuk TFP (pola EQ-1 sudah ada, tinggal adopt).


## Previous State (per 2026-05-19 — CNSD Recorder Meter Reading module shipped)

### Perubahan Terbaru

| Fitur | Status |
|---|---|
| Card "Recorder" diaktifkan di CNSD index (sebelumnya Coming Soon) | ✅ |
| Backend module: CnsdRecorderMeterRecord/Technician/Item + service/template/controller/requests | ✅ |
| Migrations: `cnsd_recorder_meter_records`, `cnsd_recorder_meter_technicians`, `cnsd_recorder_meter_items` | ✅ |
| Partial unique index per (form_type, facility, date, shift_type) | ✅ |
| Item template: 72 items (Section A: 4 groups × 69 items, Section B: 3 items, 15 U/S blocked) | ✅ |
| Form-number format: `RECORDER-YYMMDD-SEQ` (contoh `RECORDER-260517-001`) | ✅ |
| Roster integration: pull MT, Supervisor CNSD, all CNS technicians | ✅ |
| Signature authorization: name-match, immutable, no delegation, per-technician row | ✅ |
| Endpoints: 8 routes di bawah `/api/v1/cnsd/recorder-meter` | ✅ |
| Frontend list page, detail/edit page, signature panel, print view | ✅ |
| Smart input variants: Normal/Alrm dropdown (KVM), √ / - dropdown (SERVER, env), Normal/Fault dropdown (CHANNEL), free text (POWER, suhu) | ✅ |
| U/S item rendering: red strip, disabled inputs, "U/S" label, backend update guard | ✅ |
| EQ-1 dan Radar tetap berjalan (tidak ada refactor besar yang merusak) | ✅ |
| Backend tests pass (2 passed) | ✅ |
| Frontend build green | ✅ |

**File diubah/ditambah session ini:**

Backend (atoms-maintenance):
- New: `app/Models/Cnsd/CnsdRecorderMeterRecord.php`, `CnsdRecorderMeterTechnician.php`, `CnsdRecorderMeterItem.php`
- New: `app/Services/Cnsd/CnsdRecorderMeterTemplate.php`, `CnsdRecorderMeterService.php`
- New: `app/Http/Controllers/Api/V1/Cnsd/CnsdRecorderMeterController.php`
- New: `app/Http/Requests/Cnsd/{Create,Update,Sign}CnsdRecorderMeterRequest.php`
- New: `app/Exceptions/CnsdRecorderMeterDuplicateException.php`
- New: 3 migrations under `database/migrations/2026_05_19_*`
- Modified: `routes/api.php` (Recorder route group)
- Modified: `BACKEND_CONTEXT.md` (Recorder module section)

Frontend (atoms-maintenance):
- New: `src/types/cnsdRecorder.ts`
- New: `src/services/cnsdRecorderMeterService.ts`
- New: `src/pages/cnsd/CnsdRecorderMeterListPage.tsx`
- New: `src/pages/cnsd/CnsdRecorderMeterDetailPage.tsx`
- New: `src/pages/cnsd/CnsdRecorderMeterPrintView.tsx`
- New: `src/pages/cnsd/components/CnsdRecorderMeterSignaturePanel.tsx`
- Modified: `src/pages/cnsd/CnsdIndexPage.tsx` — `CNSD-003` masuk `CNSD_ACTIVE_ROUTES`
- Modified: `src/router/index.tsx` — register Recorder list/detail/print routes
- Modified: `src/data/mockData.ts` — Recorder card `is_active_mvp = true`

Documentation:
- Modified: `.agents/agents.md` — CNSD backend module count (2 → 3)
- Modified: `.agents/instructions/cnsd-readiness-rules.md` — Module 3 section
- Modified: `.kiro/steering/index.md` — Recorder Module 3 status table

**Session 2026-05-19 (Documentation Audit):**
- Created: `KIRO_TASK_CONTEXT.md` — single source of truth ringkas untuk prompt berikutnya
- Fixed: `BACKEND_CONTEXT.md` format nomor Radar (lama: `RADAR-METER-RADAR-YYYYMMDD-SEQ` → benar: `RADAR-YYMMDD-SEQ`)
- Updated: `.kiro/steering/index.md` — tambah referensi KIRO_TASK_CONTEXT.md, update date ke 2026-05-19
- Updated: `.agents/instructions/signature-system-rules.md` — tambah Radar + Recorder ke scope list + Required Signatories table
- Updated: `.agents/instructions/cnsd-readiness-rules.md` — bersihkan "What's NOT Built Yet" yang stale
- Updated: `.agents/agents.md` — tambah KIRO_TASK_CONTEXT.md ke Key Instructions, update date ke 2026-05-19

**Session 2026-05-19 (Git Commit & Push):**
- Backend commit: `b07d1c2` — `feat(cnsd): add radar and recorder meter reading modules` — author: `aryqnugroho`
- Frontend commit: `2d4067a` — `feat(cnsd): add radar and recorder meter reading UI` — author: `aryqnugroho`
- Backend pushed: `origin/main` ✅
- Frontend pushed: `origin/main` ✅
- Git author backend diperbaiki dari `AI Agent` → `aryqnugroho` (local config)
- Git author frontend sudah benar sejak awal (`aryqnugroho`)
- Commit lama yang muncul sebagai "AI Agent" (fde9929, e79ba16, dll) tidak diubah — perlu history rewrite terpisah jika diperlukan

**Audit awal session ini:**
Sebelum coding, audit menemukan kondisi inkonsisten:
- 3 migrations Recorder sudah ada dan applied (batch 6).
- 2 model Recorder (Record + Technician) sudah ada, valid.
- Item model, Template, Service, Controller, Requests, Exception, dan SEMUA frontend Recorder belum ada.
- Routes belum di-register di `routes/api.php`.
- `CnsdIndexPage` `CNSD_ACTIVE_ROUTES` map belum punya entry CNSD-003.
- `mockData.ts` Recorder card masih `is_active_mvp: false`.
Semua existing partial files dipertahankan (clean & valid), missing pieces dibangun fresh.

**Build/test:** `rtk php artisan migrate` ✅ no pending | `rtk php artisan route:list` ✅ 24 CNSD routes (8+8+8) | `rtk test php artisan test` ✅ 2 passed | `rtk npm run build` ✅ green | smoke test 72 items (15 U/S blocked at exact channels 7, 21-27, 29-35) ✅ | form number `RECORDER-260517-001` ✅

### Next Steps (Prioritas)

1. **End-to-end SSO walkthrough** untuk Recorder — perlu 4 server berjalan + akun SSO rostering. Smoke test backend ✅, manual UI test perlu user.
2. **Commit + push** session ini.
3. **Notification adoption** untuk Radar + Recorder — adopt pola yang sama dengan EQ-1.
4. **Phase 5** — modul CNSD keempat (AMSC/Transmitter/...) hanya jika ada form referensi resmi.
5. TFP, Ground Check, Grounding, Logbook, Reporting modules.

---

## Previous State (per 2026-05-18 — CNSD Radar Meter Reading refinement: Oscilator Drift + Green/Red dropdown)

### Perubahan Terbaru (Refinement)

| Item | Change | Status |
|---|---|---|
| Oscilator Drift standard `null` → `'Green'` | Template PHP + migration untuk existing records | ✅ Fixed |
| Dropdown Green/Red untuk semua item dengan `standard === 'Green'` | Frontend `TxInput` component — TX I dan TX II menggunakan `<select>` | ✅ Implemented |
| Print view: Green/Red → styled chip (hijau/merah), Antenna → "xxx RPM" | Frontend `TxPrintCell` component | ✅ Implemented |
| Backend validation tetap string (backward-compatible) | Tidak ada perubahan di `UpdateCnsdRadarMeterRequest` | ✅ No change needed |

**File diubah session ini:**

Backend:
- `app/Services/Cnsd/CnsdRadarMeterTemplate.php` — `Oscilator Drift` standard `null` → `'Green'`
- New: `database/migrations/2026_05_18_200000_fix_oscilator_drift_standard.php` — UPDATE existing records

Frontend:
- `src/pages/cnsd/CnsdRadarMeterDetailPage.tsx` — helper `isGreenStandard()`, component `TxInput`, dropdown Green/Red
- `src/pages/cnsd/CnsdRadarMeterPrintView.tsx` — helper `TxPrintCell` untuk Green/Red chip + Antenna RPM display

**Build/test:** `rtk php artisan migrate` ✅ | `rtk test php artisan test` ✅ 2 passed | `rtk npm run build` ✅ green

### Next Steps (Prioritas)

1. **End-to-end SSO walkthrough** — verifikasi manual:
   - Oscilator Drift → standard = Green, TX I/TX II dropdown
   - Semua item standard Green → dropdown Green/Red
   - Antenna, Max Deviation, Time of Revolution → tetap input manual
   - Print view → Green/Red chip, Antenna RPM
2. **Commit + push** session bugfix + refinement
3. **Notification adoption** untuk Radar (pola EQ-1 sudah ada)
4. **Phase 5** — modul CNSD ketiga

---

## Previous State (per 2026-05-18 — CNSD Radar Meter Reading bugfixes)

### Perubahan Terbaru (Bugfixes)

| Issue | Root Cause | Fix | Status |
|---|---|---|---|
| Row Antenna muncul sebagai subheader, tidak ada input RPM | Item sebelumnya di-seed dengan `item_name = '* Antenna'` (prefix `* ` = subheader marker). Template sudah diperbaiki ke `'Antenna'`, tapi data lama di DB belum diupdate. | Migration `2026_05_18_100000_fix_*` UPDATE `item_name = '* Antenna'` → `'Antenna'` | ✅ Fixed |
| Section Lingkungan Kerja hilang di UI | Template diubah dari `section_code = 'C'` ke `'B'`, tapi item di DB masih `'C'`. Frontend lookup `itemsBySection['B']` = []. | Migrasi yang sama UPDATE `section_code = 'C'` → `'B'` WHERE `section_name = 'LINGKUNGAN KERJA'` | ✅ Fixed |
| Format nomor form salah (`RADAR-METER-RADAR-YYYYMMDD-SEQ`) | `generateFormNumber()` memakai concatenation `{formType}-{facility}-{dateCompact}` | Diganti: `RADAR-{YYMMDD}-{SEQ}` → contoh `RADAR-260517-001` | ✅ Fixed |

**File diubah/ditambah session ini:**

Backend (atoms-maintenance):
- New: `database/migrations/2026_05_18_100000_fix_cnsd_radar_meter_items_section_and_antenna.php`
- Modified: `app/Services/Cnsd/CnsdRadarMeterService.php` — `generateFormNumber()` baru

Frontend: tidak ada perubahan di session bugfix ini. Semua fix di backend + migration.

**Build/test:** `rtk php artisan migrate` ✅ 1 migration (fix) | `rtk php artisan route:list` ✅ 38 routes | `rtk test php artisan test` ✅ 2 passed | `rtk npm run build` ✅ green

### Next Steps (Prioritas)

1. **End-to-end SSO walkthrough** — verifikasi manual setelah fix:
   - Buka detail Radar, tab A → Antenna harus punya input RPM TX I dan TX II
   - Buka detail Radar, tab B → 3 item Lingkungan Kerja harus tampil
   - Buat record baru → nomor form harus format `RADAR-YYMMDD-SEQ`
2. **Commit + push** perubahan session ini.
3. **Notification adoption** untuk Radar (pola EQ-1 ada, tinggal adopt).
4. **Phase 5** — modul CNSD ketiga (Recorder/AMSC) hanya jika ada form referensi resmi.

---

## Previous State (per 2026-05-18 — CNSD Radar Meter Reading module)

### Perubahan Terbaru

| Fitur | Status |
|---|---|
| Card "Radar" diaktifkan di CNSD index (sebelumnya Coming Soon) | ✅ |
| Backend module: CnsdRadarMeterRecord/Technician/Item + service/template/controller/requests | ✅ |
| Migrations: `cnsd_radar_meter_records`, `cnsd_radar_meter_technicians`, `cnsd_radar_meter_items` | ✅ |
| Partial unique index per (form_type, facility, date, shift_type) | ✅ |
| Item template: 48 items (Section A: 4 groups × ~45 items, Section C: 3 items) | ✅ |
| Form-number format: `RADAR-METER-RADAR-YYYYMMDD-SEQ` | ✅ |
| Roster integration: pull MT, Supervisor CNSD, all CNS technicians | ✅ |
| Signature authorization: name-match, immutable, no delegation, per-technician row | ✅ |
| Endpoints: `GET/POST /api/v1/cnsd/radar-meter`, `/{id}`, `/{id}/sign`, `/template`, `/years` | ✅ |
| Frontend list page, detail/edit page, signature panel, print view | ✅ |
| Dual-layout sections: TX I/TX II for section A, single HASIL column for section C | ✅ |
| EQ-1 module tetap berjalan (tidak ada refactor besar yang merusak) | ✅ |
| Backend tests pass (2 passed) | ✅ |
| Frontend build green | ✅ |

**File diubah/ditambah session ini:**

Backend (atoms-maintenance):
- New: `app/Models/Cnsd/CnsdRadarMeterRecord.php`, `CnsdRadarMeterTechnician.php`, `CnsdRadarMeterItem.php`
- New: `app/Services/Cnsd/CnsdRadarMeterTemplate.php`, `CnsdRadarMeterService.php`
- New: `app/Http/Controllers/Api/V1/Cnsd/CnsdRadarMeterController.php`
- New: `app/Http/Requests/Cnsd/{Create,Update,Sign}CnsdRadarMeterRequest.php`
- New: `app/Exceptions/CnsdRadarMeterDuplicateException.php`
- New: 3 migrations under `database/migrations/2026_05_18_*`
- Modified: `routes/api.php` (Radar route group)

Frontend (atoms-maintenance):
- New: `src/types/cnsdRadar.ts`
- New: `src/services/cnsdRadarMeterService.ts`
- New: `src/pages/cnsd/CnsdRadarMeterListPage.tsx`
- New: `src/pages/cnsd/CnsdRadarMeterDetailPage.tsx`
- New: `src/pages/cnsd/CnsdRadarMeterPrintView.tsx`
- New: `src/pages/cnsd/components/CnsdRadarMeterSignaturePanel.tsx`
- Modified: `src/pages/cnsd/CnsdIndexPage.tsx` — route map per active card code
- Modified: `src/router/index.tsx` — register Radar list/detail/print routes
- Modified: `src/data/mockData.ts` — Radar card `is_active_mvp = true`

**Build/test:** `rtk php artisan migrate` ✅ 3 new migrations | `rtk php artisan route:list` ✅ 38 routes (incl. 8 Radar) | `rtk php artisan test` ✅ 2 passed | `rtk npm run build` ✅ green | smoke test 48 items ✅

### Next Steps (Prioritas)

1. **End-to-end SSO walkthrough** untuk Radar — perlu 4 server berjalan + akun SSO rostering tersedia.
2. **Notification on create/sign-complete untuk Radar** — adopt pola yang sama dengan EQ-1 (`CnsdReadinessCreated/Completed`). Belum diimplementasikan di session ini untuk fokus pada modul Radar.
3. **Phase 5** — modul CNSD ketiga (Recorder/AMSC) hanya jika ada form referensi resmi.
4. TFP, Ground Check, Grounding, Logbook, Reporting modules.

---

## Previous State (per 2026-05-17 — CNSD Readiness Form EQ-1 + Print + Notifications + Signer Badge)

### Perubahan Terbaru

| Fitur | Status |
|---|---|
| Frontend dummy `CnsdEq1FormPage` dihapus | ✅ |
| Card "Kesiapan Peralatan CNSD" → flow baru `/cnsd/readiness` | ✅ |
| Backend CNSD readiness module (migrations, models, service, controller, routes) | ✅ |
| EQ-1 item template (5 sections, 36 items) | ✅ |
| Roster integration: pull MT, Supervisor CNSD, all CNS technicians | ✅ |
| Signature authorization: name-match, immutable, no delegation, per-technician row | ✅ |
| Partial unique index per (form_type, facility, date, shift_type) | ✅ |
| Frontend list page, detail/edit page, signature panel | ✅ |
| **`CnsdReadinessPrintView.tsx` — print layout menyerupai paper form EQ-1** | ✅ |
| **Route `/cnsd/readiness/:id/print` (outside AppShell, same pattern as WO)** | ✅ |
| **Tombol Print di list page (aksi column) + detail page (header toolbar)** | ✅ |
| **Badge "Perlu TTD" di list page untuk signer yang relevan** | ✅ |
| **`CnsdReadinessCreatedNotification` + `CnsdReadinessCompletedNotification`** | ✅ |
| **`NotificationService::notifyReadinessCreated()` + `notifyReadinessCompleted()`** | ✅ |
| Notification fired on create (graceful — non-fatal) | ✅ |
| Notification fired on completed transition during sign | ✅ |
| Backend tests pass (2 passed) | ✅ |
| Frontend build green (TypeScript clean, Vite OK) | ✅ |
| Backend pushed → `fde9929` on `main` | ✅ |
| Frontend pushed → `83d1758` on `main` | ✅ |

**File diubah/ditambah session ini:**

Backend (atoms-maintenance):
- New: `app/Notifications/CnsdReadinessCreatedNotification.php`
- New: `app/Notifications/CnsdReadinessCompletedNotification.php`
- Modified: `app/Services/NotificationService.php` (added `notifyReadinessCreated`, `notifyReadinessCompleted`)
- Modified: `app/Http/Controllers/Api/V1/Cnsd/CnsdReadinessController.php` (added `NotificationService` injection + notification calls on store + sign)

Frontend (atoms-maintenance):
- New: `src/pages/cnsd/CnsdReadinessPrintView.tsx` (print layout EQ-1)
- Modified: `src/pages/cnsd/CnsdReadinessListPage.tsx` (print icon + signer badge `Perlu TTD` / `Selesai`)
- Modified: `src/pages/cnsd/CnsdReadinessDetailPage.tsx` (Print button in header)
- Modified: `src/router/index.tsx` (route `/cnsd/readiness/:id/print`)

### Next Steps (Prioritas)

1. **End-to-end SSO walkthrough** — perlu semua 4 server berjalan dan akun SSO rostering tersedia. Checklist manual ada di TASK 5 di atas.
2. **CNSD print view refinement** — setelah user mencoba print pertama kali, mungkin ada penyesuaian layout, font size, atau spacing agar persis paper form.
3. **`local-users:sync` cron** — schedule daily sync via Laravel scheduler untuk auto-handle pegawai baru / nonaktif.
4. **Reverb / WebSocket** — realtime sign-state update di dashboard.
5. **Phase 5+** — TFP, Ground Check, Grounding, Logbook, Reporting modules.

---

## Previous State (per 2026-05-17 — CNSD Readiness Form EQ-1 pilot)

### Perubahan Terbaru

| Fitur | Status |
|---|---|
| Frontend dummy `CnsdEq1FormPage` dihapus | ✅ |
| Card "Kesiapan Peralatan CNSD" → flow baru `/cnsd/readiness` | ✅ |
| Backend migrations: `cnsd_readiness_records`, `cnsd_readiness_technicians`, `cnsd_readiness_items` | ✅ |
| Legacy CNSD scaffold tables (categories, sections, …) dropped | ✅ |
| Models: `CnsdReadinessRecord` (HasSignature), `CnsdReadinessTechnician`, `CnsdReadinessItem` | ✅ |
| `CnsdEq1Template` — 5 section, 36 baseline item dengan column header per section | ✅ |
| `CnsdReadinessService` — listRecords, createRecord, updateItems, signRecord, deleteRecord | ✅ |
| Roster integration: pull MT, Supervisor CNSD, all CNS technicians for shift+date | ✅ |
| Form number generator: `EQ-1-CNSD-YYYYMMDD-SEQ` | ✅ |
| Signature authorization: name-match, immutable, no delegation, per-technician row | ✅ |
| Endpoints: `GET/POST /api/v1/cnsd/readiness`, `/{id}`, `/{id}/sign`, `/template`, `/years` | ✅ |
| Partial unique index per (form_type, facility, date, shift_type) excluding soft-deleted | ✅ |
| Frontend: list page, detail/edit page, signature panel, sectioned table, save flow | ✅ |
| Documentation: `cnsd-readiness-rules.md` | ✅ Created |
| Print view (frontend) | ⬜ Next task — placeholder |

**File diubah/ditambah (ringkasan):**

Backend (atoms-maintenance):
- Migrations: `2026_05_17_00000{1,2,3,4}_*` (drop legacy + 3 new tables)
- Models: `app/Models/Cnsd/CnsdReadinessRecord.php`, `CnsdReadinessTechnician.php`, `CnsdReadinessItem.php`
- Service: `app/Services/Cnsd/CnsdEq1Template.php`, `app/Services/Cnsd/CnsdReadinessService.php`
- Controller: `app/Http/Controllers/Api/V1/Cnsd/CnsdReadinessController.php`
- Form Requests: `app/Http/Requests/Cnsd/{Create,Update,Sign}CnsdReadinessRequest.php`
- Exception: `app/Exceptions/CnsdReadinessDuplicateException.php`
- Routes: `routes/api.php`
- Deleted: 4 legacy empty CNSD scaffold migrations + 4 empty controllers + 4 empty models

Frontend (atoms-maintenance):
- Deleted: `src/pages/cnsd/CnsdEq1FormPage.tsx` (dummy form)
- New: `src/pages/cnsd/CnsdReadinessListPage.tsx`
- New: `src/pages/cnsd/CnsdReadinessDetailPage.tsx`
- New: `src/pages/cnsd/components/CnsdReadinessSignaturePanel.tsx`
- New: `src/services/cnsdReadinessService.ts`
- New: `src/types/cnsd.ts`
- Updated: `src/pages/cnsd/CnsdIndexPage.tsx` (active card now → `/cnsd/readiness`)
- Updated: `src/router/index.tsx` (legacy `/cnsd/eq-1` → redirect)
- Updated: `src/data/mockData.ts` (dashboard checklist route)

**Build/test:** `rtk php artisan migrate` ✅ | `rtk php artisan route:list` ✅ 30 routes (incl. 8 CNSD readiness) | `rtk php artisan test` ✅ 2 passed | `rtk npm run build` ✅ green | smoke test direct service calls ✅ all green

### Next Steps (Prioritas)

1. **EQ-1 print view** — frontend-only HTML print layout yang menyerupai paper
   form (kop AirNav Indonesia, 5 section table, footer 3 kolom signature
   teknisi/supervisor/manager). Endpoint backend tidak diperlukan kecuali
   PDF generation server-side jadi requirement nanti.
2. **Notification trigger** untuk readiness create / completed (adopt
   `NotificationService` pattern dari Work Order).
3. **Phase 5+** — CNSD card lain (Radar, Recorder, AMSC, ...) hanya jika
   sudah ada form referensi resmi. Schema sudah siap (`form_type` polymorphic).
4. TFP, Ground Check, Grounding, Logbook, Reporting modules.

---

## Previous State (per 2026-05-17 — Work Order Filter UI + WO Number Format)

### Perubahan Terbaru

| Fitur | Status |
|---|---|
| Work Order list: 5 filter baru (tanggal, tahun, divisi, shift, status) | ✅ Implemented |
| Active filter chips + Reset button | ✅ Implemented |
| Empty state (DB kosong vs filter no-result) | ✅ Implemented |
| Result count | ✅ Implemented |
| Backend `year` filter + `GET /api/v1/work-orders/years` endpoint | ✅ Implemented |
| WO number format: `WO-{DIV}-{YYYYMMDD}-{SEQ}` | ✅ Implemented |

**File diubah:**
- `frontend_atoms-maintenance/src/pages/work-order/WorkOrderListPage.tsx`
- `frontend_atoms-maintenance/src/services/workOrderService.ts`
- `backend_atoms-maintenance/app/Services/WorkOrderService.php`
- `backend_atoms-maintenance/app/Http/Controllers/Api/V1/WorkOrder/WorkOrderController.php`
- `backend_atoms-maintenance/routes/api.php`

**Build/test:** `rtk npm run build` ✅ | `rtk php artisan route:list` ✅ 22 routes | `rtk php artisan test` ✅ 2 passed

### Next Steps (Prioritas)

1. **Phase 4 — CNSD Backend module** (EQ-1 inspections). Scaffold sudah ada.
2. **Schedule `local-users:sync` harian** via Laravel scheduler.
3. **Reverb / WebSocket** untuk WO sign-status live update di dashboard list.
4. **Phase 5+** modules: TFP, Ground Check, Grounding, Logbook, Reporting.

---

## Previous State (per 2026-05-16 — UI Submit, Empty DB, local-users:sync)

### Active Module
- **Working zone:** `atoms-maintenance/`
- **Frontend:** `frontend_atoms-maintenance/` — React 19 + Vite 8, light-mode only
- **Backend:** `backend_atoms-maintenance/` — Laravel + PostgreSQL

### Service Status

| Service | Port | Status |
|---------|------|--------|
| atoms-rostering backend | 8001 | ✅ Running |
| atoms-maintenance backend | 8000 | ✅ Running |
| atoms-rostering frontend | 5174 | ✅ Running (start manually) |
| atoms-maintenance frontend | 5173 | ✅ Running (start manually) |

### Phase Status

| Phase | Module | Status |
|-------|--------|--------|
| Phase 1 | Project scaffold, DB config, mock auth | ✅ Complete |
| Phase 2 | Employee & Shift data endpoints | ✅ Complete |
| Phase 3 | Work Orders API (CRUD, sign, print) | ✅ Complete & Verified |
| Phase 3.5 | Rostering DB integration + shift personnel | ✅ Complete |
| Phase 3.6 | SSO integration (token proxy via rostering API) | ✅ Complete |
| Phase 3.7 | Roster filter (date + shift) + per-division supervisor | ✅ Complete |
| Phase 3.8 | Dashboard MT, WO create/edit, signature authz | ✅ Complete |
| Phase 3.9 | UI submit + empty DB + sync command | ✅ Complete (2026-05-16) |
| Phase 4 | CNSD Inspections (EQ-1) | ⬜ Scaffold only — next priority |
| Phase 5+ | TFP, Ground Check, Grounding, Logbook, Reporting | ⬜ Planned |

---

## Last Session Summary (2026-05-16 — UI Submit + Empty DB + sync command)

### Akar Masalah

1. **UI submit Work Order silent fail.** Backend OK via curl, tapi UI:
   - `output_types` tidak required di UI → user submit tanpa pilih output → 422 dari backend.
   - `personnel` array kosong saat roster tidak terbaca di FE → 422 (`personnel.required, min:1`).
   - Banner error muncul di top form, sering off-screen kalau user di posisi scroll bawah.
   - Tombol kadang disabled karena description < 10 char tapi user tidak melihat petunjuk.

2. **Edit submit silent fail dulu.** Sudah diperbaiki di session sebelumnya (fetch dari API), session ini hanya verifikasi ulang.

3. **Database tidak boleh ada dummy WO.** `DatabaseSeeder` masih memanggil `WorkOrderSeeder` yang membuat 4-7 WO sample, dan `MockUserSeeder` membuat 7 local_users dengan rostering_user_id yang mismatch (mis. local Moch. Ichsan claim rostering_id=2, sebenarnya 9).

4. **Tidak ada bulk sync local_users.** Lazy-create per-login dan per-WO-create sudah jalan, tapi belum ada cara seed bulk satu kali.

### Perbaikan

1. **Backend** — `WorkOrderService::autoFillShiftPersonnelFromRostering()` baru. Untuk `wo_type='shift'`, jika frontend kirim personnel kosong, backend pull dari rostering shift personnel filtered by division dan map ke local_users.id via resolver. FormRequest `personnel` jadi `sometimes` (tidak required).

2. **Frontend `WorkOrderFormModal`** — pre-validation client-side: `description.trim().length >= 10`, `outputs.length >= 1`, `output_other` wajib jika `'other'`, `selectedTechnician` wajib untuk personal. Banner error dengan `role="alert"` + `aria-live="assertive"` + `scrollIntoView({behavior:'smooth', block:'center'})` agar selalu terlihat. Output ditandai required (asterisk merah). Dev-mode console log payload summary tanpa expose token/signature.

3. **Frontend `WorkOrderListPage`** — start dengan list kosong (`useState<WorkOrder[]>([])`), bukan mock. Mock hanya dipakai sebagai fallback saat API truly unreachable. Empty list dari API sukses → tetap empty.

4. **Frontend `WorkOrderDetailPage`** — `fetchWorkOrder` di-extract sebagai `useCallback`, dipanggil di mount, di `window 'focus'` event, dan setelah signature/feedback mutation. State tidak pernah trust mutation response saja.

5. **Database seeders** — `DatabaseSeeder::run()` sekarang kosong. `MockUserSeeder` dan `WorkOrderSeeder` ditandai `@deprecated` dengan PHPDoc + runtime warning saat dipanggil. `migrate:fresh --seed` → 0 work_orders, 0 local_users.

6. **Console commands** baru:
   - `php artisan local-users:sync` — bulk pull dari rostering, support `--dry-run` & `--prune-stale`.
   - `php artisan local-users:cleanup` — deteksi & hapus duplikat stale, support `--dry-run` & `--force`. Aman: tidak hapus row yang punya FK references.

### File Diubah

Backend (atoms-maintenance):
- `app/Console/Commands/SyncLocalUsersCommand.php` — baru
- `app/Console/Commands/CleanupLocalUsersCommand.php` — baru
- `app/Services/WorkOrderService.php` — `autoFillShiftPersonnelFromRostering`
- `app/Http/Requests/WorkOrder/WorkOrderCreateRequest.php` — `personnel` jadi `sometimes`
- `database/seeders/DatabaseSeeder.php` — di-empty
- `database/seeders/MockUserSeeder.php` — deprecation warning
- `database/seeders/WorkOrderSeeder.php` — deprecation warning

Frontend (atoms-maintenance):
- `src/pages/work-order/components/WorkOrderFormModal.tsx` — pre-validation, scroll error, dev log
- `src/pages/work-order/WorkOrderListPage.tsx` — empty initial state
- `src/pages/work-order/WorkOrderDetailPage.tsx` — refetch on mount + focus + after mutation

Tidak ada perubahan di atoms-rostering. SSO/auth, JWT, localStorage, personal_access_tokens, db_rostering tetap tidak disentuh.

### Verifikasi Runtime

- `rtk npm run build` (frontend) → ✅ green
- `rtk php artisan route:list` (backend) → ✅ 21 routes intact
- `rtk php artisan test` (backend) → ✅ 2 passed
- `rtk php artisan migrate:fresh --seed` → ✅ 0 work_orders, 0 local_users
- `rtk php artisan local-users:sync` (post-fresh) → ✅ Created 54, Updated 0, Skipped 0
- `rtk php artisan local-users:sync --dry-run` (pre-fresh) → ✅ Detected 2 duplicates (Argo Pragolo, Moch. Ichsan)
- Curl create WO dengan minimal payload (no personnel) → ✅ 201, backend auto-fill 7 personnel
- Curl PUT WO update → ✅ 200, description / output_types / notes berubah, signatures preserved

### Next Steps (Prioritas)

1. **Phase 4 — CNSD Backend module** (EQ-1 inspections).
2. **Reverb / WebSocket** untuk WO sign-status live update di dashboard.
3. **`local-users:sync` cron** — schedule daily sync untuk auto-handle pegawai baru / nonaktif.
4. **Form CNSD EQ-1** — UI checklist multi-section dengan signature.
5. **Phase 5+** modules: TFP, Ground Check, Grounding, Logbook, Reporting.

---

## Key Reference Files

| Purpose | File |
|---------|------|
| **Panduan menjalankan & deploy** | **`CARA_MENJALANKAN_ATOMS_MAINTENANCE.md`** (root project) |
| Backend agent rules | [`backend_atoms-maintenance/AGENTS.md`](../atoms-maintenance/backend_atoms-maintenance/AGENTS.md) |
| Frontend agent rules | [`frontend_atoms-maintenance/AGENTS.md`](../atoms-maintenance/frontend_atoms-maintenance/AGENTS.md) |
| Backend context | [`backend_atoms-maintenance/BACKEND_CONTEXT.md`](../atoms-maintenance/backend_atoms-maintenance/BACKEND_CONTEXT.md) |
| Frontend context | [`frontend_atoms-maintenance/FRONTEND_CONTEXT.md`](../atoms-maintenance/frontend_atoms-maintenance/FRONTEND_CONTEXT.md) |
| SSO rules | [`.agents/instructions/sso-rules.md`](./instructions/sso-rules.md) |
| Work order rules | [`.agents/instructions/workorder-rules.md`](./instructions/workorder-rules.md) |
| Signature system rules | [`.agents/instructions/signature-system-rules.md`](./instructions/signature-system-rules.md) |
| Rostering integration guide | [`backend_atoms-maintenance/docs/ROSTERING_INTEGRATION.md`](../atoms-maintenance/backend_atoms-maintenance/docs/ROSTERING_INTEGRATION.md) |
| API plan | [`backend_atoms-maintenance/API_PLAN.md`](../atoms-maintenance/backend_atoms-maintenance/API_PLAN.md) |
| DB plan | [`backend_atoms-maintenance/DATABASE_PLAN.md`](../atoms-maintenance/backend_atoms-maintenance/DATABASE_PLAN.md) |
| Dev workflow | [`backend_atoms-maintenance/DEVELOPMENT_WORKFLOW.md`](../atoms-maintenance/backend_atoms-maintenance/DEVELOPMENT_WORKFLOW.md) |
