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

## Priority 3 — Perlu Manual Test

- [ ] Smoke test end-to-end: login rostering → klik Maintenance → masuk dashboard maintenance tanpa login ulang.
- [ ] Verifikasi token tersimpan di sessionStorage (bukan localStorage) via DevTools.
- [ ] Verifikasi Work Order tidak rusak.
- [ ] Verifikasi Dashboard menampilkan roster berdasarkan tanggal + shift aktif.
- [ ] Verifikasi CNSD (EQ-1, Radar, Recorder) tidak rusak.
- [ ] Verifikasi TFP (AOB Ground, AOB Lt12) tidak rusak.
- [ ] Verifikasi Signature immutable dan tidak bisa diwakilkan.
- [ ] Verifikasi Print tidak auto-print.

## Priority 4 — Opsional / Next Development

- [ ] Fix 9 test failures di atoms-rostering RosterControllerTest (test fixture issue).
- [ ] Jalankan `php artisan local-users:sync` di atoms-maintenance setelah login pertama berhasil.
- [ ] Publish roster di atoms-rostering untuk tanggal aktif agar dashboard maintenance bisa menampilkan data.
- [ ] Setup roster untuk shift pagi/siang/malam agar CNSD dan TFP bisa membuat record.
- [ ] Notification adoption untuk Radar dan Recorder (pola EQ-1 sudah ada).
- [ ] CNSD modul berikutnya (AMSC, Transmitter, dll) jika ada form referensi resmi.
- [ ] TFP modul berikutnya (AOB Lantai lain, Transmitter, dll) jika ada form referensi resmi.
