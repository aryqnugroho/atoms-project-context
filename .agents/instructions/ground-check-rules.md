# Ground Check — Module Rules

Last updated: 2026-05-19
Status: ✅ First Ground Check module live (ADC)

---

## Active Modules

| Code | Name | Form Type | Backend | Frontend | Print |
|---|---|---|---|---|---|
| GC-001 | Ground Check ADC | GC-ADC | ✅ | ✅ `/ground-check/adc` | ✅ |

Other Ground Check cards (VHF, Localizer, Glide Path, DVOR) remain **Coming Soon**.

---

## Key Difference from CNSD/TFP

| Aspek | CNSD | TFP | Ground Check ADC |
|---|---|---|---|
| Teknisi | `employee_type='CNS'` | `employee_type='Support'` | `employee_type='Support'` (TFP) |
| Supervisor | CNS grade ≥ 13 | Support grade ≥ 13 | Support grade ≥ 13 (TFP) |
| Manager Teknik | dari roster | dari roster | dari roster |
| Template | hardcoded per alat | hardcoded per alat | hardcoded per alat |
| Kolom item | varies | panel columns | TX1/TX2 (Hasil PD, In Tolerance, Out of Tolerance) |

Ground Check ADC uses **TFP personnel** (Support), not CNSD.

---

## Database

### `ground_check_adc_records`
Header per form. Satu record per (`form_type`, `date`, `shift_type`).
- Partial unique index `ground_check_adc_records_unique_form_per_shift`.
- `form_type` default `GC-ADC`.
- `report_month` — auto-derived from date (e.g., "MEI 2026").
- `airport` — default `JUANDA SURABAYA`.
- `equipment_name` — default `ADC`.
- `equipment_location` — default `GEDUNG TX, RX DAN TOWER`.
- `equipment_function` — free text (user fills).
- `technical_data` — free text (user fills).
- `last_calibration` — free text (user fills).
- `day_name` — hari Indonesia, auto dari date.
- `time_filled` — jam saat form diisi (HH:MM), refresh setiap Simpan.
- `status`: `ongoing` | `on_hold` | `completed`.
- Soft delete aktif.

### `ground_check_adc_technicians`
Snapshot teknisi TFP per record. Per-row immutable signature.

### `ground_check_adc_items`
24 items (3 headers + 2 TRANSMITTER + 2 RECEIVER + 20 CONSOLE) dari template.
Kolom: `section_name`, `item_code`, `parameter_name`, `calibration_result`, `tolerance`,
`tx1_hasil_pd`, `tx1_in_tolerance`, `tx1_out_of_tolerance`,
`tx2_hasil_pd`, `tx2_in_tolerance`, `tx2_out_of_tolerance`, `keterangan`, `is_header`.

---

## Item Template (ADC)

Sumber kebenaran: `app/Services/GroundCheck/GroundCheckAdcTemplate.php`.

3 sections:
- **TRANSMITTER** (2 items): Power, Modulation (Tx)
- **RECEIVER** (2 items): Squelch On, Audio Distorsi
- **CONSOLE** (20 items): Intercom Unit, Manual Changeover Switch, Wind Speed, Wind Direction, Temperatur, Pressure, Crass Bell Function, Headset, Hand Microphone, PTT Foot Switch, Audio Control Unit, Clock Display, Lampu meja operator, Indikator lamp, Interconnection, Antenna System, Recording, Nav Aid monitor, Interference, Hot Line

Prefilled values:
- `calibration_result`: reference standard from paper form (e.g., "30", "90", "Berfungsi", "Baik", "CLEAR")
- `tolerance`: reference tolerance (e.g., "P.Comm -3 dB", "90 ± 5%", "Clear")
- TX1/TX2 columns: always empty on new records — user fills manually

---

## Form Number Format

Format: `GC-ADC-{YYMMDD}-{SEQ}`
Contoh: `GC-ADC-260519-001`

- Prefix: `GC-ADC`
- Tanggal: YYMMDD (6 digit)
- SEQ: 3-digit zero-padded, reset per tanggal
- Counter pakai `withTrashed()` agar soft-deleted rows tetap menambah seq.

---

## Roster Integration

Saat `POST /api/v1/ground-check/adc`:

1. Backend ambil **Manager Teknik** dari `getShiftManager(shift, date)`.
2. Backend ambil **Supervisor TFP** dari `getShiftSupervisorByDivision(shift, date, 'Support')`.
3. Backend ambil **seluruh teknisi TFP** dari `getShiftPersonnel(shift, date)` filter `employee_type = 'Support'`.
4. **Personel CNSD/CNS tidak diikutkan.**
5. Jika tidak ada teknisi TFP → create gagal **422**.
6. Signer role dedup applied (supervisor/manager excluded from technician list).

---

## Signature Authorization

Identik dengan TFP:
- Role check: Manager Teknik / Supervisor TFP / Teknisi TFP.
- Nama signer harus cocok (tolerant compare).
- Teknisi sign per-row.
- Signature **immutable**, **tidak boleh diwakilkan**.

HTTP status: 200 OK, 403 nama/role tidak cocok, 409 sudah ditandatangani, 422 payload invalid.

---

## API Endpoints

Semua di bawah `/api/v1/ground-check/adc`, protected via `mockauth`:

| Method | URI | Description |
|---|---|---|
| GET    | `/api/v1/ground-check/adc` | Paginated list, filter `search`, `date`, `year`, `shift_type`, `status` |
| GET    | `/api/v1/ground-check/adc/years` | Daftar tahun tersedia |
| GET    | `/api/v1/ground-check/adc/template` | Template ADC items |
| POST   | `/api/v1/ground-check/adc` | Create. Auto-generate items + auto-attach personnel. 409 jika sudah ada. |
| GET    | `/api/v1/ground-check/adc/{id}` | Detail dengan items + technicians + signatures |
| PUT    | `/api/v1/ground-check/adc/{id}` | Update metadata + item values |
| POST   | `/api/v1/ground-check/adc/{id}/sign` | Sign role `manager` / `supervisor` / `technician` |
| DELETE | `/api/v1/ground-check/adc/{id}` | Soft-delete. Role: Admin / MT |

Print: frontend-only (`/ground-check/adc/:id/print`).

---

## Frontend Architecture

| Page | Path | File |
|---|---|---|
| Ground Check index | `/ground-check` | `pages/ground-check/GroundCheckIndexPage.tsx` |
| ADC list | `/ground-check/adc` | `pages/ground-check/GroundCheckAdcListPage.tsx` |
| ADC detail/edit | `/ground-check/adc/:id` | `pages/ground-check/GroundCheckAdcDetailPage.tsx` |
| ADC print | `/ground-check/adc/:id/print` | `pages/ground-check/GroundCheckAdcPrintView.tsx` |
| Signature panel | — | `pages/ground-check/components/GroundCheckAdcSignaturePanel.tsx` |

Type definitions: `src/types/groundCheckAdc.ts`. API client: `src/services/groundCheckAdcService.ts`.

---

## Landing Page

Ground Check landing page shows only 5 equipment cards:
1. **ADC** — Active (links to `/ground-check/adc`)
2. **VHF** — Coming Soon
3. **Localizer** — Coming Soon
4. **Glide Path** — Coming Soon
5. **DVOR** — Coming Soon

Style follows CNSD/TFP card pattern (rounded-2xl, border, code, name, location, Aktif badge).

---

## Hard Constraints

- ❌ Jangan ambil personel CNSD/CNS untuk Ground Check ADC — hanya `employee_type='Support'`.
- ❌ Jangan ubah `form_number` / `date` / `shift_type` setelah tersimpan.
- ❌ Jangan biarkan teknisi A menandatangani row milik teknisi B.
- ❌ Jangan prefill kolom TX1/TX2 — harus kosong saat record baru.
- ✅ Item template hanya boleh diubah via `GroundCheckAdcTemplate` source.
- ✅ Personnel selalu di-resolve dari roster, tidak dari mock data.
- ✅ CNSD, TFP, Work Order, Grounding tetap berjalan tanpa perubahan.
