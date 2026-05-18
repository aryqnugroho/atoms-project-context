# SSO Integration Rules — atoms-maintenance

Last updated: 2026-05-15
Status: ✅ Implemented

---

## Scope

File ini mendefinisikan aturan arsitektur untuk SSO integration antara atoms-rostering dan atoms-maintenance.
Implementasi sudah selesai. Aturan ini berlaku untuk semua task yang menyentuh auth flow.

---

## Pendekatan yang Dipilih: Token Proxy via API

atoms-maintenance memvalidasi Sanctum token dengan memanggil endpoint `GET /api/auth/me` milik atoms-rostering.
**Tidak ada shared database coupling. Tidak ada JWT.**

### Alur Token

```
atoms-rostering frontend (port 5174)
  → User klik tombol Maintenance
  → MenuGrid.tsx baca token dari sessionStorage
  → window.location.href = 'http://localhost:5173?token={sanctum_token}'

atoms-maintenance frontend (port 5173)
  → AuthContext.initAuth() baca ?token dari URL
  → Hapus token dari URL (history.replaceState)
  → Panggil GET /api/v1/auth/verify dengan token sebagai Bearer header
  → Backend proxy ke GET http://localhost:8001/api/auth/me
  → Jika valid: simpan token di sessionStorage, set user di context
  → Jika invalid: redirect ke http://localhost:5174/login
```

---

## Aturan Wajib

### Token Storage
- **Gunakan `sessionStorage`** — bukan `localStorage`
- Key: `auth_token` (konsisten di kedua frontend)
- Session berakhir saat tab browser ditutup — ini behavior yang benar untuk delegated auth

### Token Validation
- **Jangan query `personal_access_tokens` langsung** dari `atoms_rostering` DB
- **Selalu proxy** ke `GET http://localhost:8001/api/auth/me`
- Jika rostering backend down → return 401, jangan fallback ke DB query

### Token Format
- Sanctum opaque token: `{id}|{random_string}`
- URL-encode dengan `encodeURIComponent()` saat passing via URL query param
- **Jangan gunakan JWT** — bukan pilihan arsitektur ini

### Mock Dev Mode
- `DEV_MOCK_AUTH=true` (backend) + `VITE_DEV_MOCK_AUTH=true` (frontend) → SSO di-bypass
- Mock login di `http://localhost:5173/login` dengan `dudik@airnav.co.id` (password apapun)
- Mock form di `LoginPage.tsx` harus dipertahankan — jangan dihapus
- Untuk test SSO real: set kedua flag ke `false`

---

## File-File SSO (Jangan Diubah Tanpa Alasan Kuat)

| File | Peran |
|------|-------|
| `app/Services/RosteringAuthService.php` | HTTP proxy ke rostering `/api/auth/me` |
| `app/Http/Middleware/VerifyRosteringToken.php` | Standalone middleware (alias: `rostering.auth`) |
| `app/Http/Middleware/MockAuthMiddleware.php` | Handle mock + prod path |
| `app/Http/Controllers/Api/V1/AuthController.php` | `verify()` endpoint publik |
| `routes/api.php` | `GET /api/v1/auth/verify` (public route) |
| `bootstrap/app.php` | Alias `rostering.auth` |
| `frontend/src/contexts/AuthContext.tsx` | SSO token-from-URL flow |
| `frontend/src/components/layout/ProtectedRoute.tsx` | Production redirect ke rostering |
| `frontend/src/pages/auth/LoginPage.tsx` | Production redirect + mock form |
| `frontend/src/services/authService.ts` | `verify()` method + sessionStorage |
| `atoms-rostering/frontend_atoms/src/components/feature/home/MenuGrid.tsx` | Maintenance button SSO redirect |

---

## Role Mapping (Rostering → Maintenance)

| Rostering Role | Maintenance Role | Catatan |
|---------------|-----------------|---------|
| `Admin` | `Admin` | Direct map |
| `Manager Teknik` | `Manager Teknik` | Direct map |
| `General Manager` | `Manager Teknik` | Closest equivalent |
| `Cns` | `Teknisi CNSD` | Supervisor upgrade via grade check (grade ≥ 13) — belum diimplementasi |
| `Support` | `Teknisi TFP` | Supervisor upgrade via grade check (grade ≥ 13) — belum diimplementasi |

### Supervisor Upgrade Logic (Pending)
`Cns` employees dengan `grade >= 13` adalah supervisor di konteks maintenance.
Logic ini belum diimplementasi di `RosteringAuthService::mapRole()`.
Harus ditambahkan saat CNSD supervisor-specific routes diimplementasi.

---

## Token Payload dari Rostering `/api/auth/me`

| Field | Type | Deskripsi |
|-------|------|-----------|
| `id` | int | rostering user ID (= `local_users.rostering_user_id`) |
| `name` | string | Nama lengkap |
| `email` | string | Email |
| `role` | string | `Admin`, `Cns`, `Support`, `Manager Teknik`, `General Manager` |
| `grade` | int\|null | Grade karyawan (13+ = supervisor level untuk CNS) |
| `is_active` | bool | Flag akun aktif |
| `employee.id` | int | Employee record ID |
| `employee.employee_type` | string | `CNS`, `Support`, `Manager Teknik`, `Administrator` |
| `employee.group_number` | int\|null | Nomor group/divisi |
| `employee.is_fixed_manager` | bool | Flag fixed manager |

---

## Known Limitations

1. **Tidak ada token refresh:** Sanctum token di atoms-rostering tidak punya expiry. Jika token di-revoke di rostering (logout), atoms-maintenance akan return 401 pada request berikutnya. Frontend akan redirect ke rostering login.

2. **atoms-rostering harus running:** Jika rostering backend down, atoms-maintenance return 401 untuk semua request. `RosteringAuthService` log warning dan fail closed.

3. **Supervisor role distinction:** Belum diimplementasi. Semua `Cns` saat ini di-map ke `Teknisi CNSD`.

---

## Cara Test SSO

1. Start semua 4 service (lihat `ROSTERING_INTEGRATION.md` Section 5 startup order)
2. Buka `http://localhost:5174`, login sebagai `admin@airnav.com` / `password`
3. Klik card Maintenance
4. Konfirmasi redirect ke `http://localhost:5173?token=...`
5. Konfirmasi token dihapus dari URL dan dashboard load
6. Konfirmasi nama/role user di top bar sesuai akun rostering
7. Konfirmasi semua API call include `Authorization: Bearer {token}` header

---

## Referensi Lengkap

Lihat `backend_atoms-maintenance/docs/ROSTERING_INTEGRATION.md` Section 7 untuk detail implementasi lengkap.
