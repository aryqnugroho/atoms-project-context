# Manual Form Development Hook — ATOMS MAINTENANCE

> **Baca file ini sebelum membuat atau memperbaiki form manual apapun di ATOMS MAINTENANCE.**
> Berlaku untuk: CNSD, TFP, Ground Check, Grounding, dan semua form manual berikutnya.
> Last updated: 2026-05-19

---

## 1. Tujuan Hook

Hook ini adalah checklist wajib untuk pembuatan form manual baru di ATOMS MAINTENANCE.

**Prinsip utama:**
- Semua form manual mengikuti pola **CNSD EQ-1 / Kesiapan Peralatan** sebagai referensi utama.
- **Dynamic form masih HOLD** — jangan diimplementasikan dalam kondisi apapun.
- Setiap form baru harus konsisten dari backend, frontend, print view, roster, signature, testing, dokumentasi, commit, hingga push.
- Jangan menebak struktur file — selalu baca modul yang sudah ada sebagai referensi aktual.

**Form yang sudah ada sebagai referensi:**

| Domain | Form | Referensi Utama |
|--------|------|-----------------|
| CNSD | EQ-1 Kesiapan Peralatan | ✅ Referensi utama semua pola |
| CNSD | Radar Meter Reading | ✅ Pola meter reading dual TX |
| CNSD | Recorder Meter Reading | ✅ Pola U/S blocked items |
| CNSD | AMSC Meter Reading | ✅ Pola channel + time_filled |
| TFP | AOB Lantai Ground | ✅ Pola disabled cells + facilities |
| TFP | AOB Lantai 1 & 2 | ✅ Pola single-value panel columns |

---

## 2. File yang Wajib Dibaca Sebelum Mulai

Baca semua file berikut sebelum menulis satu baris kode pun:

**Selalu:**
- `KIRO_TASK_CONTEXT.md` — hard constraints, SSO, signature, CNSD/TFP state
- `PROGRESS_LOG.md` jika ada
- `NEXT_TASKS.md` jika ada
- `.agents/session-handoff.md` — current state dan next steps
- `atoms-maintenance/backend_atoms-maintenance/BACKEND_CONTEXT.md`

**Untuk task CNSD:**
- `.agents/instructions/cnsd-readiness-rules.md`
- Modul referensi: EQ-1, Radar, Recorder, AMSC (baca file aktual, bukan asumsi)
- Print view EQ-1 (`CnsdReadinessPrintView.tsx`) sebagai referensi footer signature

**Untuk task TFP:**
- `.agents/instructions/tfp-rules.md`
- Modul referensi: AOB Ground, AOB Lt 1 & 2 (baca file aktual)
- Print view Recorder (`CnsdRecorderMeterPrintView.tsx`) sebagai referensi footer

**Untuk task Ground Check / Grounding:**
- Baca modul CNSD/TFP yang paling mirip sebagai referensi pola

---

## 3. Pull / Sync Wajib Sebelum Coding

**Jangan mulai coding sebelum semua repo up to date.**

Jalankan di masing-masing repo:

```bash
# Backend
rtk git status
rtk git branch --show-current
rtk git remote -v
rtk git pull --rebase origin main
rtk git status

# Frontend
rtk git status
rtk git branch --show-current
rtk git remote -v
rtk git pull --rebase origin main
rtk git status

# Context (root ATOMS-PROJECT)
rtk git status
rtk git branch --show-current
rtk git pull --rebase origin main
rtk git status
```

**Aturan pull:**
- Jangan force pull.
- Jangan overwrite uncommitted changes.
- Jika ada local changes, jalankan `rtk git diff` dan laporkan ke user.
- Jika local changes adalah pekerjaan valid, stash dulu: `rtk git stash`, pull, lalu `rtk git stash pop`.
- Jangan mulai coding sebelum backend dan frontend up to date.

---

## 4. Hard Constraints

Semua constraint ini berlaku tanpa pengecualian:

| Constraint | Keterangan |
|-----------|------------|
| ❌ Jangan ubah SSO/auth | SSO sudah berjalan, jangan disentuh |
| ❌ Jangan buat login baru | Login ada di atoms-rostering |
| ❌ Jangan gunakan JWT | Hanya Sanctum opaque token |
| ❌ Jangan simpan token di localStorage | Wajib sessionStorage |
| ❌ Jangan query `personal_access_tokens` langsung | Proxy via `/api/auth/me` |
| ❌ Jangan write ke `db_rostering` | Read-only |
| ❌ Jangan merusak Work Order | Modul stabil, jangan disentuh |
| ❌ Jangan merusak modul CNSD/TFP yang sudah berjalan | Setiap modul hidup di namespace terpisah |
| ❌ Dynamic form HOLD | Jangan implementasikan dalam kondisi apapun |
| ❌ Jangan force push | Selalu pull --rebase jika remote lebih baru |
| ✅ Semua terminal command wajib RTK | Lihat `.kiro/steering/rtk-rules.md` |
| ✅ Jangan push sebelum build/test hijau | Build dan test harus pass dulu |

---

## 5. Pola Backend Form Manual

Untuk setiap form baru, buat file-file berikut (ikuti pola modul yang sudah ada):

### Models
```
app/Models/{Domain}/
  {FormName}Record.php       — header record, HasSignature + SoftDeletes
  {FormName}Technician.php   — per-row technician snapshot + signature
  {FormName}Item.php         — item rows dari template
  {FormName}Facility.php     — jika ada checklist fasilitas (pola TFP)
```

### Services
```
app/Services/{Domain}/
  {FormName}Template.php     — template item hardcoded, sumber kebenaran
  {FormName}Service.php      — CRUD, roster, sign, form number
```

### Controller
```
app/Http/Controllers/Api/V1/{Domain}/
  {FormName}Controller.php
```

### Requests
```
app/Http/Requests/{Domain}/
  Create{FormName}Request.php
  Update{FormName}Request.php
  Sign{FormName}Request.php
```

### Exception
```
app/Exceptions/
  {FormName}DuplicateException.php
```

### Migration
```
database/migrations/
  YYYY_MM_DD_NNNNNN_create_{table_name}_records_table.php
  YYYY_MM_DD_NNNNNN_create_{table_name}_technicians_table.php
  YYYY_MM_DD_NNNNNN_create_{table_name}_items_table.php
  YYYY_MM_DD_NNNNNN_create_{table_name}_facilities_table.php  (jika ada)
```

### Routes (tambahkan di `routes/api.php`)
```
GET    /api/v1/{domain}/{form-code}
GET    /api/v1/{domain}/{form-code}/years
GET    /api/v1/{domain}/{form-code}/template
POST   /api/v1/{domain}/{form-code}
GET    /api/v1/{domain}/{form-code}/{id}
PUT    /api/v1/{domain}/{form-code}/{id}
POST   /api/v1/{domain}/{form-code}/{id}/sign
DELETE /api/v1/{domain}/{form-code}/{id}
```

Semua route harus protected dengan middleware `mockauth` yang sama.
`/template` dan `/years` harus dideklarasikan **sebelum** `/{id}` agar tidak tertangkap parameter.

### Partial Unique Index
Setiap tabel records harus punya partial unique index PostgreSQL:
```sql
CREATE UNIQUE INDEX {table}_unique_form_per_shift
ON {table} (form_type, facility, date, shift_type)
WHERE deleted_at IS NULL
```
Ini mencegah duplikat record aktif per tanggal + shift, tapi membolehkan re-create setelah soft-delete.

---

## 6. Pola Frontend Form Manual

Untuk setiap form baru, buat:

```
src/types/{formName}.ts                          — type definitions
src/services/{formName}Service.ts                — API client
src/pages/{domain}/
  {FormName}ListPage.tsx                         — list + filter + create
  {FormName}DetailPage.tsx                       — detail/edit + section tabs
  {FormName}PrintView.tsx                        — print view manual only
  components/
    {FormName}SignaturePanel.tsx                 — signature panel
```

**Update yang diperlukan:**
- `src/router/index.tsx` — tambah routes list/detail/print
- `src/pages/{domain}/{Domain}IndexPage.tsx` — aktifkan card di `{DOMAIN}_ACTIVE_ROUTES`
- `src/data/mockData.ts` — set `is_active_mvp: true` untuk card

**Flow user:**
1. User klik card di index page
2. Masuk list page (search + filter + tombol Buat)
3. Klik Buat → backend generate record + template + roster
4. User edit isi form di detail page (section tabs)
5. User sign sesuai hak di signature panel
6. User klik Print → print view terbuka, print manual

---

## 7. Pola Roster

**Wajib:** ambil roster berdasarkan **tanggal + shift**, bukan tanggal saja.

### CNSD
```php
$manager    = $rosteringService->getShiftManager($shiftType, $date);
$supervisor = $rosteringService->getShiftSupervisorByDivision($shiftType, $date, 'CNS');
$personnel  = $rosteringService->getShiftPersonnel($shiftType, $date)
                ->filter(fn($p) => $p->employee_type === 'CNS');
```
- Teknisi = seluruh CNS pada tanggal + shift
- Supervisor = CNS dengan grade ≥ 13 (nullable)
- Manager Teknik = dari roster tanggal + shift (nullable)
- TFP/Support **tidak diikutkan**

### TFP
```php
$manager    = $rosteringService->getShiftManager($shiftType, $date);
$supervisor = $rosteringService->getShiftSupervisorByDivision($shiftType, $date, 'Support');
$personnel  = $rosteringService->getShiftPersonnel($shiftType, $date)
                ->filter(fn($p) => $p->employee_type === 'Support');
```
- Teknisi = seluruh Support pada tanggal + shift
- Supervisor = Support dengan grade ≥ 13 (nullable)
- Manager Teknik = dari roster tanggal + shift (nullable)
- CNSD/CNS **tidak diikutkan**

### Aturan Roster
- Jangan menebak mapping employee_type — verifikasi dari service roster aktual.
- Cache nama personel saat record dibuat (immutable setelah create).
- Jangan ambil ulang nama dari rostering untuk record lama.
- `db_rostering` read-only — jangan pernah write.
- Jika tidak ada teknisi → create gagal **422** dengan pesan jelas.
- Supervisor nullable — jika tidak ada, kolom null, tanda tangan tidak diwajibkan.
- Manager nullable — jika tidak ada, kolom null, tanda tangan tidak diwajibkan.

---

## 8. Pola Signature

### Aturan Dasar
- Signature = base64 PNG data URL (`data:image/png;base64,...`)
- **Immutable** — setelah tersimpan tidak bisa diubah atau dihapus
- **Tidak bisa diwakilkan** — signer harus user yang namanya tercatat di record
- Validasi utama di backend — frontend hanya UX hint (disable tombol)

### HTTP Status
| Kondisi | Status |
|---------|--------|
| Signature tersimpan | 200 |
| Role tidak cocok / nama tidak cocok | 403 |
| Sudah pernah ditandatangani | 409 |
| Payload tidak valid (bukan PNG base64) | 422 |

### Aturan Per Role
- **Teknisi:** hanya sign baris miliknya sendiri (match by `technician_id` atau name match)
- **Supervisor:** hanya sign dirinya sendiri (name match)
- **Manager:** hanya sign dirinya sendiri (name match)
- Name match: tolerant — trim + collapse whitespace + case-insensitive

### Footer Print
Footer signature print view harus mengikuti pola **Form Kesiapan/EQ-1**:
```
[Waktu Pelaksanaan: Hari | Tanggal | Jam]   ← jika form punya time_filled
┌─────────────────────┬──────────────┬──────────────────┐
│ TEKNISI             │ SUPERVISOR   │ MANAGER TEKNIK   │
│ No | Nama | Paraf   │ [signature]  │ [signature]      │
│ 1  | ...  | [TTD]   │ atau         │ atau             │
│ 2  | ...  | [TTD]   │ dashed box   │ dashed box       │
│ ...                 │ atau         │ atau             │
│                     │ "Tidak ada   │ "Tidak           │
│                     │  supervisor" │  ditugaskan"     │
└─────────────────────┴──────────────┴──────────────────┘
```
- Gunakan `border border-black` (bukan `border-dashed border-gray-400`) untuk konsistensi
- Tampilkan `signed_at` timestamp jika sudah TTD
- Tampilkan "Belum TTD" italic jika belum TTD
- Tampilkan "Tidak ada supervisor pada shift ini" jika supervisor null
- Tampilkan "Manager Teknik tidak ditugaskan" jika manager null

---

## 9. Pola Nomor Form

Format umum: `PREFIX-YYMMDD-SEQ`

| Form | Format | Contoh |
|------|--------|--------|
| EQ-1 | `EQ-1-CNSD-YYYYMMDD-SEQ` | `EQ-1-CNSD-20260519-001` |
| Radar | `RADAR-YYMMDD-SEQ` | `RADAR-260519-001` |
| Recorder | `RECORDER-YYMMDD-SEQ` | `RECORDER-260519-001` |
| AMSC | `AMSC-YYMMDD-SEQ` | `AMSC-260519-001` |
| TFP AOB Ground | `TFP-AOBLTGND-YYMMDD-SEQ` | `TFP-AOBLTGND-260519-001` |
| TFP AOB Lt 1&2 | `TFP-AOBLT12-YYMMDD-SEQ` | `TFP-AOBLT12-260519-001` |

**Aturan:**
- SEQ = 3-digit zero-padded, reset per tanggal
- Counter pakai `withTrashed()` agar soft-deleted rows tetap menambah sequence
- Tampil di list, detail, dan print view
- Jangan ubah `form_number` setelah tersimpan
- Ikuti format spesifik yang diberikan user jika berbeda dari pola umum

---

## 10. Pola Template Item

- Template item harus di **backend** (`app/Services/{Domain}/{FormName}Template.php`)
- Frontend hanya render data dari API — jangan hardcode dummy data di frontend
- Template adalah sumber kebenaran — perubahan item hanya via edit file template
- Jika ada dropdown, definisikan dari nominal/context di template
- Jika ada cell disabled/abu-abu, backend harus tahu dan menolak update (`is_disabled_map`)
- Jika ada U/S items, tetap tampil dengan red strip — jangan dihapus
- Keterangan biasanya free text
- Print view boleh lebih document-like daripada edit page

**Kolom item yang umum dipakai:**

| Kolom | Dipakai Untuk |
|-------|---------------|
| `nominal` | Expected value dari paper form |
| `hasil_a`, `hasil_b` | Dual column (TX I/II, Server A/B, HASIL A/B) |
| `hasil` | Single result column |
| `address`, `status_value`, `cct` | Channel AMSC |
| `kondisi_teknis_tx1`, `kondisi_teknis_tx2` | Radar TX dual |
| `hasil_server_a`, `hasil_server_b` | Recorder Server dual |
| `is_blocked`, `block_reason` | U/S items |
| `is_disabled_map` (jsonb) | TFP disabled cells |
| `keterangan` | Free text notes |

---

## 11. Pola UI Detail/Edit

- Detail/edit harus user-friendly — tidak harus 100% identik dengan print
- Gunakan warna dan font konsisten ATOMS (sky/slate palette)
- Untuk form kompleks, gunakan **section tabs** (Front Panel | Power Supply | Channel | Lingkungan)
- Jangan tampilkan teks tidak perlu seperti "Mengikuti format form resmi"
- Disabled cell harus jelas secara visual (`bg-slate-300` atau `bg-slate-200`)
- Dropdown harus jelas — gunakan `<select>` dengan opsi yang tepat
- Untuk tabel dengan banyak kolom, gunakan `overflow-x-auto`
- Header info card (tanggal, shift, lokasi, teknisi) di atas tabel

**Dropdown standar berdasarkan nominal:**

| Nominal | Dropdown Options |
|---------|-----------------|
| `Normal / Alrm` | Normal, Alrm |
| `√ / -` | √, - |
| `OK / Not` | OK, Not |
| `√` | √, - |
| `Green` (Radar) | Green, Red |
| Channel status | Normal, U/S, Fault |
| Kondisi fasilitas | Baik, Normal, Tidak Baik |
| Mode | Auto, Manual |
| Suplai Aktif | PLN / UPS atau PLN 1 / PLN 2 |

---

## 12. Pola Print View

**Wajib:**
- Gunakan logo AirNav asli: `/assets/icon/logoairnav.svg`
- Header/kop mengikuti form resmi (logo kiri, Perum LPPNPI kanan)
- Footer signature mengikuti pola Form Kesiapan/EQ-1 (lihat bagian 8)
- **Print manual only** — jangan auto-print saat halaman dibuka

**Larangan auto-print:**
```tsx
// ❌ JANGAN — auto-print saat halaman dibuka
useEffect(() => {
  setTimeout(() => window.print(), 500);
}, [record]);

// ✅ BENAR — print hanya saat klik tombol
<button onClick={() => window.print()}>Print PDF</button>
```

**CSS print yang wajib:**
```tsx
<style>
  {`
    @media print {
      @page { size: A4 portrait; margin: 8mm 10mm; }
      body { background: white !important; }
      .print-hide { display: none !important; }
      tr, td { page-break-inside: avoid; }
    }
  `}
</style>
```

**Toolbar (screen only):**
```tsx
<div className="print-hide mx-auto mb-4 flex max-w-[210mm] items-center justify-between">
  <Button variant="outline" onClick={() => navigate(`/{domain}/{form}/${record.id}`)}>
    <ArrowLeft size={16} /> Kembali
  </Button>
  <Button onClick={() => window.print()}>
    <Printer size={16} /> Print PDF
  </Button>
</div>
```

**Waktu pelaksanaan di footer (jika form punya `time_filled`):**
```tsx
<div className="flex border-b border-black text-[10px]">
  <div className="flex-1 grid grid-cols-[70px_1fr] p-2 border-r border-black">
    <span className="font-bold">Hari</span>
    <span>: {record.day_name ?? '-'}</span>
  </div>
  <div className="flex-1 grid grid-cols-[70px_1fr] p-2 border-r border-black">
    <span className="font-bold">Tanggal</span>
    <span>: {formatDate(record.date)}</span>
  </div>
  <div className="flex-1 grid grid-cols-[50px_1fr] p-2">
    <span className="font-bold">Jam</span>
    <span>: {record.time_filled ?? '-'}</span>
  </div>
</div>
```

---

## 13. Testing Wajib

Jalankan semua command berikut setelah implementasi selesai:

```bash
# Backend
rtk php artisan migrate                    # jika ada migration baru
rtk php artisan route:list --path={domain} # verifikasi 8 routes terdaftar
rtk test php artisan test                  # semua test harus pass

# Frontend
rtk npm run build                          # TypeScript harus clean, build harus hijau

# Status
rtk git diff                               # review perubahan
rtk git status                             # pastikan tidak ada file tidak terduga
```

**Manual test checklist:**
1. Login via atoms-rostering
2. Masuk atoms-maintenance
3. Buka page domain (CNSD/TFP)
4. Pastikan card aktif (bukan Coming Soon)
5. Klik card → masuk list page
6. Klik Buat → record terbuat dengan form number yang benar
7. Verifikasi teknisi sesuai divisi (CNS untuk CNSD, Support untuk TFP)
8. Verifikasi supervisor dan manager teknik dari roster
9. Isi beberapa nilai → Simpan → refresh → data tersimpan
10. Sign sesuai user login → berhasil
11. Sign user tidak sesuai → harus 403
12. Buka print view → **tidak auto-print**
13. Klik Print PDF → dialog muncul **sekali**
14. Logo AirNav tampil di print view
15. Footer Hari/Tanggal/Jam tampil (jika form punya `time_filled`)
16. Signature tampil jika sudah TTD, placeholder jika belum
17. Modul lain (Work Order, CNSD lain, TFP lain) tidak rusak

---

## 14. Dokumentasi Wajib

Setelah task selesai, update semua file berikut:

| File | Yang Diupdate |
|------|---------------|
| `.agents/session-handoff.md` | Current State terbaru (perubahan, file diubah, build/test, next steps) |
| `.agents/instructions/cnsd-readiness-rules.md` | Jika task CNSD — tambah section modul baru |
| `.agents/instructions/tfp-rules.md` | Jika task TFP — tambah section modul baru |
| `BACKEND_CONTEXT.md` | Tambah section modul baru (tabel, endpoints, format nomor, roster, constraints) |
| `KIRO_TASK_CONTEXT.md` | Update tabel status modul, format nomor form jika ada yang baru |
| `.kiro/steering/index.md` | Update Integration Status table |

**Aturan dokumentasi:**
- Dokumentasi harus current state ringkas, bukan catatan mentah
- Jangan duplikasi informasi yang sudah ada di file lain
- Setiap modul baru harus punya entry di semua file di atas

---

## 15. Commit dan Push Wajib Setelah Aman

Setelah build/test hijau dan manual test aman:

### Cek sebelum commit
```bash
# Di masing-masing repo
rtk git status
rtk git diff
rtk git branch --show-current
rtk git remote -v
rtk git config user.name    # harus: aryqnugroho
rtk git config user.email
```

Jika `user.name` bukan `aryqnugroho`:
```bash
git config user.name "aryqnugroho"
```

### Stage hanya file relevan
```bash
# Backend — jangan commit .env, vendor, logs, cache
rtk git add app/Models/... app/Services/... app/Http/... database/migrations/... routes/api.php BACKEND_CONTEXT.md

# Frontend — jangan commit node_modules, dist, .env
rtk git add src/types/... src/services/... src/pages/... src/router/index.tsx src/data/mockData.ts

# Context — jangan commit atoms-maintenance/ atau atoms-rostering/
rtk git add KIRO_TASK_CONTEXT.md .agents/session-handoff.md .agents/instructions/... .kiro/steering/index.md
```

### Commit message format
```bash
# Backend
rtk git commit -m "feat(cnsd): add [form name] module"
rtk git commit -m "feat(tfp): add [form name] module"
rtk git commit -m "fix(cnsd): refine [form name] print view"

# Frontend
rtk git commit -m "feat(cnsd): add [form name] pages and print view"
rtk git commit -m "fix(cnsd): align [form name] print footer with readiness form"

# Context
rtk git commit -m "docs: update progress for [form name]"
rtk git commit -m "docs: add manual form development hook"
```

### Push
```bash
rtk git push origin main
```

Jika push gagal karena remote lebih baru:
```bash
rtk git pull --rebase origin main
# selesaikan konflik jika ada
rtk test php artisan test   # jalankan ulang test
rtk npm run build           # jalankan ulang build
rtk git push origin main
```

**Jangan force push tanpa izin eksplisit dari user.**

---

## 16. Output Akhir Standar

Setiap task form harus menghasilkan output yang mencakup:

1. Hasil pull/rebase awal (backend, frontend, context)
2. Struktur backend yang dibuat/diubah (models, services, controller, migrations, routes)
3. Endpoint API (8 routes)
4. Template form (sections, groups, items, total count)
5. Route frontend (list, detail, print)
6. Pola print view (kop, tabel, footer)
7. Roster integration (employee_type filter, nullable fields)
8. Signature authorization (role check, name match, HTTP status)
9. Format nomor form
10. Hasil build/test (`rtk php artisan migrate`, `rtk test php artisan test`, `rtk npm run build`)
11. Hasil manual test (checklist dari bagian 13)
12. Dokumentasi yang diupdate (file + ringkasan perubahan)
13. Commit hash backend
14. Commit hash frontend
15. Commit hash context
16. Status push (backend/frontend/context)
17. Hal yang belum selesai jika ada

---

## Referensi Cepat — File Aktual yang Perlu Dibaca

Sebelum membuat form baru, baca file-file ini sebagai referensi pola aktual:

### Backend (baca yang paling mirip dengan form baru)
```
app/Services/Cnsd/CnsdRecorderMeterTemplate.php   — pola template item
app/Services/Cnsd/CnsdRecorderMeterService.php    — pola service (create, roster, sign)
app/Services/Cnsd/CnsdAmscMeterService.php        — pola service dengan time_filled
app/Http/Controllers/Api/V1/Cnsd/CnsdRecorderMeterController.php  — pola controller
database/migrations/2026_05_19_000001_create_cnsd_recorder_meter_records_table.php  — pola migration
```

### Frontend (baca yang paling mirip dengan form baru)
```
src/types/cnsdRecorder.ts                         — pola type definitions
src/services/cnsdRecorderMeterService.ts          — pola API client
src/pages/cnsd/CnsdRecorderMeterListPage.tsx      — pola list page
src/pages/cnsd/CnsdRecorderMeterDetailPage.tsx    — pola detail/edit page
src/pages/cnsd/CnsdRecorderMeterPrintView.tsx     — pola print view
src/pages/cnsd/components/CnsdRecorderMeterSignaturePanel.tsx  — pola signature panel
src/pages/cnsd/CnsdReadinessPrintView.tsx         — referensi footer signature
```
