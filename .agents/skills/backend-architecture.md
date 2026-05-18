# SKILL: Backend Architecture — atoms-maintenance

> **Canonical location (Kiro):**
> `atoms-maintenance/backend_atoms-maintenance/.kiro/skills/backend-architecture/SKILL.md`
>
> This file is a **summary pointer** untuk non-Kiro AI tools (Copilot, Claude, dll).
> Jika ada konflik, canonical `.kiro/skills/` file yang berlaku.

---

Selalu ikuti aturan ini saat bekerja di ATOMS-Maintenance backend.

## Mandatory First Steps

1. **Baca AGENTS.md dan BACKEND_CONTEXT.md** sebelum memulai task backend apapun.
2. **Baca DATABASE_PLAN.md dan API_PLAN.md** untuk modul yang relevan.
3. **Cek frontend types** di `frontend_atoms-maintenance/src/types/index.ts` untuk memahami expected data shapes.
4. **Baca session-handoff.md** untuk mengetahui status task terakhir.

## Architecture Rules

1. **Two-system model.** atoms-rostering adalah source of truth untuk login, accounts, dan employee/shift data.
2. **Jangan modifikasi atoms-rostering.** Zero exceptions (kecuali `MenuGrid.tsx` untuk SSO button).
3. **Standalone API.** Laravel + PostgreSQL. No HTML views.
4. **Controller → Service pattern.** Thin controllers, logic di `app/Services/`.
5. **Form Request validation.** Semua endpoint menggunakan dedicated Form Request classes.
6. **Standard JSON responses.** `{ success, message, data, errors }` di semua endpoint.
7. **API versioning.** Semua routes di bawah `/api/v1/`.
8. **Rostering DB connection.** Gunakan `DB::connection('rostering')` untuk read-only queries. Jangan pernah write.

## Auth & SSO Rules

1. **Jangan gunakan JWT.** Auth menggunakan Laravel Sanctum opaque token.
2. **Jangan query `personal_access_tokens` langsung.** Validasi token via `GET http://localhost:8001/api/auth/me`.
3. **SSO sudah implemented.** Gunakan `RosteringAuthService` untuk token validation. Jangan buat auth baru.
4. **Mock dev mode:** `DEV_MOCK_AUTH=true` di `.env` untuk bypass SSO. Mock form di `LoginPage.tsx` harus dipertahankan.
5. **Middleware alias:** `mockauth` untuk semua protected routes. `rostering.auth` untuk explicit SSO validation.

## Key Services (Jangan Duplikasi)

| Service | File | Fungsi |
|---------|------|--------|
| `RosteringAuthService` | `app/Services/RosteringAuthService.php` | HTTP proxy ke rostering `/api/auth/me` |
| `RosteringIntegrationService` | `app/Services/RosteringIntegrationService.php` | Semua rostering DB queries (shift, personnel, supervisor) |
| `WorkOrderService` | `app/Services/WorkOrderService.php` | Work Order business logic |
| `NotificationService` | `app/Services/NotificationService.php` | Notification dispatch |

## Signature System Rules

- Semua signature disimpan sebagai base64 PNG di database (long text field). Bukan file upload.
- Gunakan `HasSignature` trait untuk semua modul yang butuh signature.
- Signature immutable setelah disimpan — jangan buat overwrite logic.
- Lihat `.agents/instructions/signature-system-rules.md` untuk detail lengkap.

## Security Rules

1. Jangan hardcode secrets — selalu gunakan `.env` variables.
2. Hanya commit `.env.example`.
3. RBAC middleware di setiap protected route.
4. Jangan simpan token di localStorage — gunakan sessionStorage (frontend concern, tapi jangan encourage sebaliknya).

## Documentation Rules

Update `API_PLAN.md`, `DATABASE_PLAN.md`, `DEVELOPMENT_WORKFLOW.md` saat schema atau routes berubah.
