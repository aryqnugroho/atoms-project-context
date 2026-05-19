# NEXT TASKS - ATOMS PROJECT

## Priority 1 — Selesai ✅

- ✅ Clone/pull repo atoms-rostering frontend dan backend.
- ✅ Clone/pull repo atoms-maintenance frontend dan backend.
- ✅ Setup env lokal masing-masing project.
- ✅ Jalankan backend atoms-rostering dan seed akun login.
- ✅ Jalankan backend atoms-maintenance.
- ✅ Jalankan frontend atoms-rostering dan frontend atoms-maintenance.

## Priority 2 — Selesai ✅

- ✅ Integrasi login atoms-rostering ke atoms-maintenance.
- ✅ Token Sanctum opaque token disimpan di sessionStorage.
- ✅ Backend maintenance validasi token ke /api/auth/me backend rostering.
- ✅ DEV_MOCK_AUTH=false (SSO mode aktif).
- ✅ Fix: card Maintenance di atoms-rostering sekarang redirect ke atoms-maintenance dengan SSO token.
- ✅ **TFP-003 Transmitter TX module live** — card aktif, form create/edit/detail/print/sign berjalan.
- ✅ **Reporting / Laporan Kerusakan module live (2026-05-19)** — Topbar menu Reporting, format LTK-YYMMDD-SEQ, manager + pelaksana dipilih manual.
- ✅ **Role-Based Signature Delegation (2026-05-19)** — semua 22 signature panel + 19 backend service diupdate.
- ✅ **Landing Page Logbook (2026-05-19)** — 2 card navigasi CNSD & TFP, routing setup.

## Priority 3 — Task Berikutnya

- [ ] **Pembuatan fitur Print View/Export PDF Logbook TFP.**

- [ ] Smoke test end-to-end di browser:
  1. Buka http://localhost:5174
  2. Login dengan admin@airnav.com / password
  3. Klik card "Maintenance" di home
  4. Verifikasi redirect ke http://localhost:5173?token=... lalu URL berubah ke http://localhost:5173/dashboard
  5. Verifikasi tidak ada login ulang di maintenance
  6. Buka DevTools → Application → Session Storage → pastikan auth_token ada di localhost:5173
  7. Pastikan auth_token TIDAK ada di localStorage
- [ ] Verifikasi Work Order tidak rusak.
- [ ] Verifikasi Dashboard menampilkan roster berdasarkan tanggal + shift aktif.
- [ ] Verifikasi CNSD (EQ-1, Radar, Recorder) tidak rusak.
- [ ] Verifikasi TFP (AOB Ground, AOB Lt12) tidak rusak.
- [ ] Verifikasi Signature immutable dan tidak bisa diwakilkan.
- [ ] Verifikasi Print tidak auto-print.
- [ ] Reporting end-to-end test:
  1. Klik menu Reporting di topbar.
  2. Klik Tambah Laporan.
  3. Isi form Section 1-5 (Data Laporan, Detail Kerusakan, Tindakan Perbaikan, Waktu Kerusakan, Kode Hambatan).
  4. Pilih Manager Teknik dari dropdown manager.
  5. Tambahkan beberapa pelaksana dari personel CNSD/TFP (campur).
  6. Pilih kode hambatan AL → pastikan field Alasan Lain muncul dan wajib.
  7. Simpan laporan, verifikasi nomor surat format LTK-YYMMDD-001.
  8. Coba sign manager dengan user yang bukan manager → harus 403.
  9. Sign dengan user yang sesuai jika tersedia.
  10. Buka print view, pastikan tidak auto muncul.
  11. Verifikasi logo AirNav tampil di header.
  12. Pastikan judul: LAPORAN KERUSAKAN.
  13. Klik Print PDF, dialog print muncul sekali.

## Priority 4 — Opsional / Next Development

- [ ] Push commit `94a5730` di atoms-rostering/frontend_atoms ke remote (menunggu konfirmasi).
- [ ] Fix 9 test failures di atoms-rostering RosterControllerTest (test fixture issue).
- [ ] Jalankan `php artisan local-users:sync` di atoms-maintenance setelah login pertama berhasil.
- [ ] Publish roster di atoms-rostering untuk tanggal aktif agar dashboard maintenance bisa menampilkan data.
- [ ] Setup roster untuk shift pagi/siang/malam agar CNSD dan TFP bisa membuat record.
- [ ] Notification adoption untuk Radar dan Recorder (pola EQ-1 sudah ada).
- [ ] CNSD modul berikutnya (AMSC, Transmitter, dll) jika ada form referensi resmi.
- [ ] TFP modul berikutnya (AOB Lantai lain, Transmitter, dll) jika ada form referensi resmi.
