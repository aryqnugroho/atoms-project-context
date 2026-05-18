# SKILL: Frontend Architecture — atoms-maintenance

> **Canonical location (Kiro):**
> `atoms-maintenance/frontend_atoms-maintenance/.kiro/skills/frontend-architecture/SKILL.md`
>
> This file is a **summary pointer** untuk non-Kiro AI tools (Copilot, Claude, dll).
> Jika ada konflik, canonical `.kiro/skills/` file yang berlaku.

---

Selalu ikuti aturan ini saat bekerja di ATOMS-Maintenance frontend.

## Mandatory First Steps

1. **Baca AGENTS.md dan FRONTEND_CONTEXT.md** sebelum memulai task frontend apapun.
2. **Baca CLAUDE.md** untuk component inventory dan design rules.
3. **Cek existing components** di `src/components/common/` sebelum membuat yang baru.
4. **Baca session-handoff.md** untuk mengetahui status task terakhir.

## Architecture Rules

1. **React 19 + TypeScript + Vite 8 + Tailwind CSS 3.4.**
2. **Light-mode only.** Jangan reintroduce dark theme.
3. **Types in one place.** Semua interfaces di `src/types/index.ts`.
4. **Mock data in one place.** Dev data di `src/data/mockData.ts`.
5. **Routes in one place.** Register di `src/router/index.tsx`.
6. **Axios over Fetch.** Gunakan `axios` dengan `VITE_API_URL` base URL.
7. **Jangan modifikasi atoms-rostering.** Zero exceptions (kecuali `MenuGrid.tsx` untuk SSO button).

## Auth & SSO Rules

1. **Jangan simpan token di localStorage.** Gunakan `sessionStorage` — key: `auth_token`.
2. **SSO sudah implemented.** `AuthContext.tsx` sudah handle token-from-URL flow. Jangan buat auth baru.
3. **Mock dev mode:** `VITE_DEV_MOCK_AUTH=true` di `.env` untuk bypass SSO. Mock form di `LoginPage.tsx` harus dipertahankan — jangan dihapus.
4. **Production redirect:** Jika tidak ada token dan bukan mock mode, redirect ke `VITE_ROSTERING_FRONTEND_URL`.
5. **Token format:** Sanctum opaque `{id}|{random}` — URL-encode dengan `encodeURIComponent()` saat passing via URL.

## Key Files (Jangan Duplikasi Logic)

| File | Fungsi |
|------|--------|
| `src/contexts/AuthContext.tsx` | SSO token-from-URL flow, user state |
| `src/components/layout/ProtectedRoute.tsx` | Route guard + production redirect |
| `src/pages/auth/LoginPage.tsx` | Mock login form (dev) + production redirect |
| `src/services/authService.ts` | `verify()` method + sessionStorage management |
| `src/services/workOrderService.ts` | Work Order API calls + `getShiftContext()` |

## Division Color Coding

- **CNSD:** Sky blue (`sky-*`)
- **TFP:** Emerald green (`emerald-*`)

## Signature System Rules

- Signature dikirim sebagai base64 PNG data URL dari canvas.
- Jangan buat file upload untuk signature.
- Gunakan `SignatureCanvas` dan `SignatureDisplay` components yang sudah ada.
- Lihat `.agents/instructions/signature-system-rules.md` untuk detail lengkap.

## Ground Check Rule

- Ground Check adalah digital logbook — form entry langsung.
- Jangan buat file upload logic untuk Ground Check.

## Validation

Jalankan `npm run lint` dan `npm run build` sebelum commit.
