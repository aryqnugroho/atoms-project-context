# CNSD Equipment Readiness — Module Rules

Last updated: 2026-05-19
Status: ✅ Three pilot modules live (Phase 4)

---

## Active Modules

| Code | Name | Form Type | Backend | Frontend | Print |
|---|---|---|---|---|---|
| CNSD-001 | Kesiapan Peralatan CNSD | EQ-1 | ✅ | ✅ `/cnsd/readiness` | ✅ |
| CNSD-002 | Radar Meter Reading | RADAR-METER | ✅ | ✅ `/cnsd/radar-meter` | ✅ |
| CNSD-003 | **Recorder Meter Reading** | **RECORDER-METER (FORM C-3)** | ✅ | ✅ `/cnsd/recorder-meter` | ✅ |
| CNSD-004 | **AMSC Meter Reading** | **AMSC-METER** | ✅ | ✅ `/cnsd/amsc-meter` | ✅ |
| CNSD-005 | **Transmitter Meter Reading** | **TRANSMITTER-METER (FORM C-1)** | ✅ | ✅ `/cnsd/transmitter-meter` | ✅ |
| CNSD-006 | **Receiver Meter Reading** | **RECEIVER-METER (FORM C-2)** | ✅ | ✅ `/cnsd/receiver-meter` | ✅ |

All other CNSD cards (Glide Path, Localizer,
T-DME, DVOR, DME, ATC System, ATIS) remain **Coming Soon** and have no
backend.

All three modules apply to fasilitas CNSD (technicians filtered by
employee_type=CNS). Each lives in its own table and namespace; refactor or
extension to one module never affects the others.

---

## Scope (legacy section retained for clarity)

Modul **Kesiapan Peralatan CNSD — Form EQ-1** adalah modul pilot CNSD pertama
yang berbackend penuh. Form EQ-1 hanya untuk fasilitas CNSD. Tidak ada flow TFP
di tabel ini.

---

## Database

### `cnsd_readiness_records`
Header per form. Satu record per (`form_type`, `facility`, `date`, `shift_type`).
- Constraint unik dijaga oleh **partial unique index PostgreSQL**
  `cnsd_readiness_records_unique_form_per_shift` yang ignore soft-deleted rows.
- `form_type` default `EQ-1`. `facility` default `CNSD`.
- `status`: `ongoing` | `on_hold` | `completed` (mengikuti pola Work Order).
- `manager_id`, `supervisor_id` nullable. Diisi otomatis dari roster saat create.
- `manager_name`, `supervisor_name`, signature columns mirror Work Order.
- Soft delete aktif.

### `cnsd_readiness_technicians`
Snapshot teknisi CNSD yang bertugas pada shift. Satu row per teknisi. Setiap row
punya signature kolom sendiri yang **immutable** dan **tidak boleh diwakilkan**.

### `cnsd_readiness_items`
Item form yang digenerate dari template EQ-1 saat record dibuat. Kolom generic
(`kondisi_operasional_1`, `kondisi_operasional_2`) supaya satu schema bisa
melayani section dengan header berbeda (SERVER AKTIF/DUAL STATE,
DUAL STATUS/FREQUENCY, TX OPERASI/DUAL STATE, CHANNEL AKTIF/DUAL STATE,
SERVER STATE/WORKSTATION STATE).

---

## Item Template (EQ-1)

Sumber kebenaran: `app/Services/Cnsd/CnsdEq1Template.php`.

5 section, total 36 item baseline:

| Section | Header kolom 1 | Header kolom 2 | Item count |
|---|---|---|---|
| KOMUNIKASI PENERBANGAN | SERVER AKTIF | DUAL STATE | 4 |
| RADIO | DUAL STATUS | FREQUENCY | 20 (banyak primary/secondary) |
| NAVIGASI PENERBANGAN | TX OPERASI | DUAL STATE | 5 |
| PENGAMATAN PENERBANGAN | CHANNEL AKTIF | DUAL STATE | 4 |
| AUTOMASI PENERBANGAN | SERVER STATE | WORKSTATION STATE | 3 |

Item ditambah/dirombak hanya melalui edit `CnsdEq1Template`. Tidak ada UI untuk
menambah/menghapus item — itu konvensi disengaja: form EQ-1 adalah dokumen
operasional yang formatnya distandarkan oleh AirNav.

Frekuensi RADIO di-prefill dari paper form (121,65 MHz untuk CDU PRIMARY, dst).
User boleh meng-override jika sewaktu-waktu frekuensi resmi berubah.

---

## Roster Integration

Saat `POST /api/v1/cnsd/readiness`:

1. Backend ambil **Manager Teknik** dari `RosteringIntegrationService::getShiftManager(date, shift)`.
2. Backend ambil **Supervisor CNSD** dari `getShiftSupervisorByDivision(date, shift, 'CNS')`.
3. Backend ambil **seluruh teknisi CNSD** dari `getShiftPersonnel(date, shift)`
   yang `employee_type = 'CNS'`. **Personel TFP/Support tidak diikutkan.**
4. Setiap personel di-resolve via `LocalUserResolver::ensureLocalUser()` agar
   row local_users dibuat lazy bila belum ada.
5. Nama setiap personel di-cache di record (immutable signer name).

Aturan filter:
- **Wajib** filter berdasarkan tanggal **dan** shift secara eksplisit.
- Tidak ambil per tanggal saja.
- Tidak campur shift lain.
- Hanya `employee_type = 'CNS'` yang masuk.
- Jika roster belum dipublish → backend mengembalikan **422** dengan pesan
  `Tidak ada teknisi CNSD yang bertugas pada tanggal X shift Y. Pastikan roster sudah dipublish ...`.
  Create gagal supaya tidak tercipta record kosong.
- Jika supervisor CNSD tidak ada di shift → `supervisor_id`/`supervisor_name`
  null, tanda tangan supervisor **tidak diwajibkan**, kolom kosong di print view.
- Jika manager teknik tidak ada → `manager_id`/`manager_name` null, tanda
  tangan manager tidak diwajibkan. Behavior ini sengaja permisif karena tidak
  semua shift punya MT bertugas. UI menampilkan label "Tidak ditugaskan".

`db_rostering` adalah read-only. Service tidak pernah menulis ke rostering.

---

## Signature Authorization

Identik dengan Work Order, dengan satu tambahan: teknisi sign per-row.

Backend `CnsdReadinessService::signRecord()`:
- Role harus cocok (Manager Teknik / Supervisor CNSD / Teknisi CNSD).
- Nama signer harus cocok dengan nama yang di-cache di record/row
  (tolerant compare: trim + collapse whitespace + case-insensitive lewat
  `WorkOrderService::namesMatch()`).
- Untuk teknisi:
  - Klien bisa mengirim `technician_row_id` eksplisit.
  - Jika tidak, backend resolve dari `signer.id` lalu fallback ke nama match.
  - Hanya row yang nama-nya cocok dengan signer yang boleh ditandatangani.
- Signature **immutable**: setelah ada nilai non-null, re-sign throw 409.
- Re-sign attempt pada record `completed` → 409.

HTTP status mapping:
- `200` — signature tersimpan.
- `403` — role tidak cocok atau nama tidak cocok (`SignerNotAuthorizedException`).
- `409` — sudah pernah ditandatangani atau record completed.
- `422` — payload tidak valid (bukan PNG base64).

Tidak boleh diwakilkan — meskipun teknisi A dan B sama-sama Teknisi CNSD,
B tidak boleh menandatangani baris milik A.

Validasi utama di backend. Frontend boleh disable tombol sebagai UX hint
(disable bila role/nama tidak cocok), tapi backend tetap menjadi gate.

---

## Status Lifecycle

Sama seperti Work Order:
- `ongoing` — masih ada signature wajib yang kosong dan shift belum berakhir.
- `on_hold` — masih ada signature wajib yang kosong **dan** shift sudah lewat.
- `completed` — semua signature wajib sudah lengkap (manager, supervisor jika
  ada, **dan setiap teknisi** sudah sign rownya masing-masing).

Status di-derive dari signature, tidak boleh diset manual oleh client.

---

## API Endpoints

Semua di bawah `/api/v1/cnsd/readiness`, protected via `mockauth` (sama dengan
Work Order — SSO via rostering). Authorization dilakukan di controller / service.

| Method | URI | Description |
|---|---|---|
| GET    | `/api/v1/cnsd/readiness` | Paginated list, filter `search`, `date`, `year`, `shift_type`, `status` |
| GET    | `/api/v1/cnsd/readiness/years` | Daftar tahun tersedia (descending) |
| GET    | `/api/v1/cnsd/readiness/template` | EQ-1 section + item template untuk preview FE |
| POST   | `/api/v1/cnsd/readiness` | Create record. Auto-generate items + auto-attach personnel. 409 jika sudah ada (mengembalikan `existing_record`). Role: Admin / Manager / Supervisor CNSD / Teknisi CNSD |
| GET    | `/api/v1/cnsd/readiness/{id}` | Detail dengan items + technicians + signatures |
| PUT    | `/api/v1/cnsd/readiness/{id}` | Update **hanya item values** (status_peralatan, kondisi_operasional_*, keterangan). Personnel & signature & metadata tidak boleh diubah |
| POST   | `/api/v1/cnsd/readiness/{id}/sign` | Sign role `manager` / `supervisor` / `technician`. Untuk technician kirim `technician_row_id` (opsional — backend bisa resolve dari signer ID/nama) |
| DELETE | `/api/v1/cnsd/readiness/{id}` | Soft-delete record. Role: Admin / Manager Teknik |

**Print endpoint:** belum dibuat di backend untuk Phase 4. Print direncanakan
sebagai **frontend-only** (HTML print view dengan media query `print:`),
mengikuti pola Work Order. Endpoint backend hanya akan dibuat jika nanti
diperlukan PDF generation server-side.

---

## Form Number Format

Format: `EQ-1-CNSD-YYYYMMDD-SEQ`
Contoh: `EQ-1-CNSD-20260517-001`

- `SEQ` = 3-digit zero-padded, reset per tanggal + form_type + facility.
- Sequence menggunakan `withTrashed()` count, jadi nomor tetap monotonik bahkan
  setelah soft-delete.

---

## Frontend Architecture

| Page | Path | File |
|---|---|---|
| CNSD index | `/cnsd` | `pages/cnsd/CnsdIndexPage.tsx` |
| Readiness list | `/cnsd/readiness` | `pages/cnsd/CnsdReadinessListPage.tsx` |
| Readiness detail/edit | `/cnsd/readiness/:id` | `pages/cnsd/CnsdReadinessDetailPage.tsx` |
| **Readiness print view** | **`/cnsd/readiness/:id/print`** | **`pages/cnsd/CnsdReadinessPrintView.tsx`** |
| Signature panel (component) | — | `pages/cnsd/components/CnsdReadinessSignaturePanel.tsx` |

Route print berada di luar `AppShell` (tidak ada topbar/sidebar) — identik dengan `WorkOrderPrintView`.

Old route `/cnsd/eq-1` sekarang **redirect** ke `/cnsd/readiness` agar bookmark lama tidak rusak.

Type definitions: `src/types/cnsd.ts`. API client: `src/services/cnsdReadinessService.ts`.

## Notifications

`NotificationService` di backend di-extended dengan dua method baru:

| Method | Trigger | Recipients |
|---|---|---|
| `notifyReadinessCreated(record)` | Saat POST `/api/v1/cnsd/readiness` berhasil | Manager, Supervisor, semua teknisi (exclude creator) |
| `notifyReadinessCompleted(record, completedBy)` | Saat status → `completed` setelah sign | Creator, Manager, Supervisor, semua teknisi (exclude signer yang memicu completion) |

Notification class: `CnsdReadinessCreatedNotification`, `CnsdReadinessCompletedNotification`.
Keduanya menggunakan channel `database` (in-app notification bell), sama seperti Work Order.
Failure non-fatal — jika notifikasi gagal (misal teknisi belum punya local_users entry), response API tetap 200/201.

## Signer Badge (List Page)

`CnsdReadinessListPage` menampilkan kolom "Anda" dengan badge berbasis role + nama:
- **Perlu TTD** (amber) — user login adalah signer yang relevan dan record belum `completed`.
- **Selesai** (emerald) — record sudah `completed` dan user adalah signer (inferred dari nama match).
- (kosong) — user bukan signer relevan pada record ini.

Logic: client-side name match terhadap `manager_name`, `supervisor_name`, `technician_names[]` dari summary API. Tidak ada per-row detail fetch. Backend tetap authority untuk validasi signature sebenarnya.

---

## What's NOT Built Yet (EQ-1)

- Reverb/WebSocket realtime untuk sign-state.
- Notification untuk Radar dan Recorder (pola EQ-1 sudah ada, tinggal adopt).

---

## Signer Role Deduplication Rule

**Rule:** Supervisor CNSD dan Manager Teknik tidak boleh masuk ke daftar Teknisi CNSD.

**Fix:** Setelah membangun daftar `$technicians` di `resolveRosterContext()`, panggil:
```php
$technicians = \App\Services\WorkOrderService::excludeSignerRoles(
    $technicians,
    $rosterSupervisor ? (int) $rosterSupervisor->user_id : null,
    $supervisor?->name,
    $rosterManager ? (int) $rosterManager->user_id : null,
    $manager?->name,
);
```

Berlaku untuk semua modul CNSD: EQ-1, Radar, Recorder, AMSC, Transmitter, dan form berikutnya.

---

## Hard Constraints

- ❌ Jangan tambah backend untuk card CNSD lain di session yang sama.
- ❌ Jangan biarkan UI menulis ke field signature secara langsung.
- ❌ Jangan ubah `cnsd_readiness_*` schema tanpa migration baru.
- ❌ Jangan write ke `db_rostering`.
- ❌ Jangan ambil personel dari shift lain atau divisi lain (TFP).
- ❌ Jangan ubah `form_number` setelah tersimpan.
- ❌ Jangan biarkan teknisi A menandatangani row milik teknisi B.
- ✅ Item template hanya boleh diubah via `CnsdEq1Template` source.
- ✅ Personnel selalu di-resolve dari roster, tidak dari mock data.


---

## Module 2: Radar Meter Reading (Form RADAR-METER) — 2026-05-18

### Database
- `cnsd_radar_meter_records` — header, satu record per (form_type, facility, date, shift_type).
  - Field Radar-spesifik: `merk` (default `ELDIS`), `type` (default `MSSR-1 / RL2000`),
    `serial_number` (nullable, "-" jika kosong).
  - Partial unique index `cnsd_radar_meter_records_unique_form_per_shift`.
- `cnsd_radar_meter_technicians` — per-row technician signature.
- `cnsd_radar_meter_items` — 48 items hasil generate template:
  - `section_code` (A/C), `section_name`, `group_number`, `group_name`, `item_number`,
    `item_name`, `standard`, `kondisi_teknis_tx1`, `kondisi_teknis_tx2`, `hasil`,
    `keterangan`, `sort_order`.

### Item Template
Sumber: `app/Services/Cnsd/CnsdRadarMeterTemplate.php`.

| Section | Layout | Groups | Items |
|---|---|---|---|
| A. TERMONITOR DI LCMS | TX I / TX II | 1: MSSR TRANSMITTER A/B, 2: SSR EXTRACTOR, 3: AILAN CH.A/CH.B, 4: RADAR CONTROL | 45 items (incl. sub-headers) |
| C. LINGKUNGAN KERJA | HASIL only | — | 3 items |

Field usage per section:
- **Section A** (TX dual): user mengisi `kondisi_teknis_tx1` (kolom TX I) dan
  `kondisi_teknis_tx2` (kolom TX II). `standard` adalah threshold dari paper
  form (mis. ">2500 W", "Green", "5,6", "+- 0").
- **Section C** (environment): user hanya mengisi `hasil`. Standar mis.
  "Max 22°C", "√".

Item dengan prefix `* ` di `item_name` adalah sub-header (mis. `* GPS Receiver`,
`* Antenna`, `* RADAR`). Frontend merender baris ini sebagai label group, bukan
input editable. Standard untuk sub-header bisa null.

### Form Number
Format baru: `RADAR-{YYMMDD}-{SEQ}`
Contoh: `RADAR-260517-001`, `RADAR-260517-002`

- Prefix: `RADAR`
- Tanggal: YYMMDD (6 digit: 2-digit tahun + 2-digit bulan + 2-digit hari)
- SEQ: 3-digit zero-padded, reset per tanggal
- Data lama dengan format `RADAR-METER-RADAR-YYYYMMDD-SEQ` tetap bisa tampil (search ILIKE di backend)
- Record baru setelah 2026-05-18 wajib memakai format baru

### Roster Integration
Identik dengan EQ-1: ambil MT, Supervisor CNSD (CNS grade ≥ 13), seluruh teknisi
CNS dari roster tanggal+shift. **Tidak ambil TFP.** Jika tidak ada teknisi CNSD
→ create gagal 422.

### API Endpoints

Semua di bawah `/api/v1/cnsd/radar-meter`, protected via `mockauth`:

| Method | URI | Description |
|---|---|---|
| GET    | `/api/v1/cnsd/radar-meter` | Paginated list, filter `search`, `date`, `year`, `shift_type`, `status` |
| GET    | `/api/v1/cnsd/radar-meter/years` | Daftar tahun tersedia |
| GET    | `/api/v1/cnsd/radar-meter/template` | Template Radar (sections + groups + items) |
| POST   | `/api/v1/cnsd/radar-meter` | Create. Auto-generate items + auto-attach personnel. 409 jika sudah ada. Role: Admin / MT / Sup CNSD / Teknisi CNSD |
| GET    | `/api/v1/cnsd/radar-meter/{id}` | Detail dengan items + technicians + signatures |
| PUT    | `/api/v1/cnsd/radar-meter/{id}` | Update item values: `kondisi_teknis_tx1/tx2`, `hasil`, `keterangan` |
| POST   | `/api/v1/cnsd/radar-meter/{id}/sign` | Sign role `manager` / `supervisor` / `technician` |
| DELETE | `/api/v1/cnsd/radar-meter/{id}` | Soft-delete. Role: Admin / MT |

Print: frontend-only (`/cnsd/radar-meter/:id/print`).

### Frontend Architecture (Radar)

| Page | Path | File |
|---|---|---|
| Radar list | `/cnsd/radar-meter` | `pages/cnsd/CnsdRadarMeterListPage.tsx` |
| Radar detail/edit | `/cnsd/radar-meter/:id` | `pages/cnsd/CnsdRadarMeterDetailPage.tsx` |
| Radar print | `/cnsd/radar-meter/:id/print` | `pages/cnsd/CnsdRadarMeterPrintView.tsx` |
| Signature panel | — | `pages/cnsd/components/CnsdRadarMeterSignaturePanel.tsx` |

Type definitions: `src/types/cnsdRadar.ts`. API client: `src/services/cnsdRadarMeterService.ts`.

### What's NOT Built Yet (Radar)
- Notification trigger saat create / completed (pola EQ-1 sudah ada, tinggal adopt).
- Backend untuk card CNSD ketiga (Recorder/AMSC/...) — masih Coming Soon.

### Known Bugs Fixed (2026-05-18)

| Bug | Cause | Fix |
|---|---|---|
| Antenna tampil sebagai header, tidak ada input RPM | item_name `'* Antenna'` di DB lama, frontend `isSubHeader()` filter `startsWith('* ')` | Migration UPDATE `'* Antenna'` → `'Antenna'` |
| Section Lingkungan Kerja kosong | section_code `'C'` di DB, tapi frontend lookup `'B'` dari sectionMeta | Migration UPDATE `section_code 'C'` → `'B'` WHERE LINGKUNGAN KERJA |
| Format nomor form salah | `generateFormNumber()` menghasilkan `RADAR-METER-RADAR-YYYYMMDD-SEQ` | Diganti: `RADAR-YYMMDD-SEQ` |

### Hard Constraints (Radar)
- ❌ Jangan ubah skema `cnsd_radar_meter_*` tanpa migration baru.
- ❌ Jangan ambil personel TFP atau dari shift lain.
- ❌ Jangan ubah `merk` / `type` / `form_number` setelah tersimpan.
- ❌ Jangan biarkan teknisi A menandatangani row teknisi B.
- ✅ Item template hanya boleh diubah via `CnsdRadarMeterTemplate` source.
- ✅ EQ-1 module tetap berjalan tanpa perubahan — Radar hidup di tabel & namespace terpisah.


---

## Radar Meter Reading — UI Refinement (2026-05-18)

### Oscilator Drift Standard
Item `Oscilator Drift` (Group 2 SSR EXTRACTOR, Section A) sekarang memiliki `standard = 'Green'` (sebelumnya null).
Existing records diupdate via migration `2026_05_18_200000_fix_oscilator_drift_standard`.

### Dropdown Green/Red untuk Item Standard "Green"
Semua item Radar Meter Reading pada Section A yang memiliki `standard === 'Green'` (perbandingan exact/case-sensitive) menggunakan dropdown **Green** / **Red** untuk kolom TX I dan TX II pada detail/edit page.

**Rule frontend:** `isGreenStandard(item) = item.standard === 'Green'`

- Green → emerald dropdown (bg-emerald-50 text-emerald-700)
- Red → red dropdown (bg-red-50 text-red-700)

**Item yang TETAP manual input** (standard bukan persis "Green"):
- Antenna → `'10 RPM / 15 RPM'` (input teks, placeholder `... RPM`)
- Power SUM / Power OMEGA → `'>2500 W'`
- VSWR SUM / VSWR OMEGA → `null`
- Max Deviation / Min Deviation → `'+- 0'`
- Time of Revolution → `'5,6'`
- All Indicators → `'OK / Green'` (bukan persis "Green")
- Rectification → `null`
- Pemeriksaan suhu Ruangan → `'Max 22°C'`
- Pemeriksaan Air Humidity / Kebersihan → `'√'`

### Print View
Komponen `TxPrintCell` merender:
- `Green` → chip hijau inline
- `Red` → chip merah inline
- Antenna → nilai RPM + " RPM" atau `"......... RPM"` placeholder
- Lainnya → teks biasa via `val()`


---

## Module 3: Recorder Meter Reading (Form RECORDER-METER, FORM C-3) — 2026-05-19

### Database
- `cnsd_recorder_meter_records` — header, satu record per (form_type, facility, date, shift_type).
  - Field Recorder-spesifik: `form_code` (default `FORM C-3`), `merk` (default `ATIS - UHER`),
    `type` (default `VC - MDx`), `serial_number` (default `51`).
  - Partial unique index `cnsd_recorder_meter_records_unique_form_per_shift`.
- `cnsd_recorder_meter_technicians` — per-row technician signature.
- `cnsd_recorder_meter_items` — 72 items hasil generate template:
  - `section_code` (A/B), `section_name`, `group_number`, `group_name`, `item_number`,
    `item_name`, `nominal`, `hasil_server_a`, `hasil_server_b`, `hasil`,
    `keterangan`, `is_blocked`, `block_reason`, `sort_order`.

### Item Template
Sumber: `app/Services/Cnsd/CnsdRecorderMeterTemplate.php`.

| Section | Layout | Groups | Items |
|---|---|---|---|
| A. PERALATAN | Server A / Server B | 1: KVM (1), 2: SERVER (2), 3: POWER (2), 4: CHANNEL (64) | 69 items |
| B. LINGKUNGAN KERJA | HASIL only | — | 3 items |

Total: **72 items**, di antaranya **15 U/S blocked** (Channel 7, 21–27, 29–35).

Field usage per section:
- **Section A** (Server dual): user mengisi `hasil_server_a` (Server A) dan
  `hasil_server_b` (Server B). `nominal` adalah expected value dari paper form
  ("Normal / Alrm", "√ / -", "Ground Primary", dst).
- **Section B** (environment): user hanya mengisi `hasil`. Nominal misalnya
  `< 22° C`, `√`.

### U/S (Un-Serviceable) Items
Beberapa channel pada paper form dicoret merah dengan label "U/S" (Un-Serviceable):
Channel 7, 21–27, 29–35.

Aturan:
- Backend seed `is_blocked = true`, `block_reason = 'U/S'`.
- Backend `updateItems()` **mengabaikan** patch yang menargetkan blocked rows.
- Frontend menampilkan baris merah dengan label U/S, semua input disabled.
- Print view menampilkan strip merah bertulisan U/S sesuai paper form.
- Item blocked tetap muncul di list — tidak dihapus.
- Tidak boleh dihitung sebagai signature requirement (signature ada di header,
  bukan per item).

### Form Number
Format: `RECORDER-{YYMMDD}-{SEQ}`
Contoh: `RECORDER-260517-001`, `RECORDER-260517-002`

- Prefix: `RECORDER`
- Tanggal: YYMMDD (6 digit)
- SEQ: 3-digit zero-padded, reset per tanggal
- Counter pakai `withTrashed()` agar soft-deleted rows tetap menambah seq.

### Roster Integration
Identik dengan EQ-1 / Radar: ambil MT, Supervisor CNSD (CNS grade ≥ 13),
seluruh teknisi CNS dari roster tanggal+shift. **Tidak ambil TFP.** Jika tidak
ada teknisi CNSD → create gagal 422.

### API Endpoints

Semua di bawah `/api/v1/cnsd/recorder-meter`, protected via `mockauth`:

| Method | URI | Description |
|---|---|---|
| GET    | `/api/v1/cnsd/recorder-meter` | Paginated list, filter `search`, `date`, `year`, `shift_type`, `status` |
| GET    | `/api/v1/cnsd/recorder-meter/years` | Daftar tahun tersedia |
| GET    | `/api/v1/cnsd/recorder-meter/template` | Template Recorder (sections + groups + items) |
| POST   | `/api/v1/cnsd/recorder-meter` | Create. Auto-generate items + auto-attach personnel. 409 jika sudah ada. Role: Admin / MT / Sup CNSD / Teknisi CNSD |
| GET    | `/api/v1/cnsd/recorder-meter/{id}` | Detail dengan items + technicians + signatures |
| PUT    | `/api/v1/cnsd/recorder-meter/{id}` | Update item values: `hasil_server_a`, `hasil_server_b`, `hasil`, `keterangan`. Blocked items diabaikan |
| POST   | `/api/v1/cnsd/recorder-meter/{id}/sign` | Sign role `manager` / `supervisor` / `technician` |
| DELETE | `/api/v1/cnsd/recorder-meter/{id}` | Soft-delete. Role: Admin / MT |

Print: frontend-only (`/cnsd/recorder-meter/:id/print`).

### Frontend Architecture (Recorder)

| Page | Path | File |
|---|---|---|
| Recorder list | `/cnsd/recorder-meter` | `pages/cnsd/CnsdRecorderMeterListPage.tsx` |
| Recorder detail/edit | `/cnsd/recorder-meter/:id` | `pages/cnsd/CnsdRecorderMeterDetailPage.tsx` |
| Recorder print | `/cnsd/recorder-meter/:id/print` | `pages/cnsd/CnsdRecorderMeterPrintView.tsx` |
| Signature panel | — | `pages/cnsd/components/CnsdRecorderMeterSignaturePanel.tsx` |

Type definitions: `src/types/cnsdRecorder.ts`. API client: `src/services/cnsdRecorderMeterService.ts`.

### Frontend Input Variants
Detail/edit page memilih input control berdasarkan `nominal` + group context:

| Nominal / Context | Input | Options |
|---|---|---|
| `Normal / Alrm` (KVM) | dropdown | `Normal`, `Alrm` |
| `√ / -` (SERVER, Lingkungan) | dropdown | `√`, `-` |
| `√` (Lingkungan humidity / kebersihan) | dropdown | `√`, `-` |
| section A group 4 (CHANNEL) — non-blocked | dropdown | `Normal`, `Fault` |
| `is_blocked = true` | disabled red strip + "U/S" label | — |
| Lainnya (POWER AC/DC, suhu ruangan `< 22° C`) | free text | — |

Color coding: `Normal`/`√` → emerald. `Alrm`/`Fault`/`-` → red.

### What's NOT Built Yet (Recorder)
- Notification trigger saat create / completed (pola EQ-1 sudah ada, tinggal adopt).
- Backend untuk card CNSD keempat (AMSC/Transmitter/...) — masih Coming Soon.

### Hard Constraints (AMSC)
- ❌ Jangan ubah skema `cnsd_amsc_meter_*` tanpa migration baru.
- ❌ Jangan ambil personel TFP atau dari shift lain.
- ❌ Jangan ubah `merk` / `type` / `serial_number` / `form_number` setelah tersimpan.
- ❌ Jangan biarkan teknisi A menandatangani row teknisi B.
- ✅ Item template hanya boleh diubah via `CnsdAmscMeterTemplate` source.
- ✅ EQ-1, Radar, dan Recorder tetap berjalan tanpa perubahan — AMSC hidup di tabel & namespace terpisah.
- ✅ `time_filled` di-refresh setiap kali user Simpan Perubahan (pola TFP AOB Ground).
- ✅ `day_name` di-set saat create dari `now()->format('w')` (Indonesian day name).
- ✅ Print view footer menampilkan Hari/Tanggal/Jam dari record, bukan dari render time.
