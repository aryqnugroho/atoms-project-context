---
inclusion: manual
---

# RTK Rules — ATOMS PROJECT

> **Baca file ini sebelum menjalankan terminal command apapun.**
> RTK adalah CLI proxy yang memfilter output command sebelum masuk ke LLM context.
> Menggunakan RTK menghemat 60–90% token pada command yang menghasilkan output besar.

---

## Apa itu RTK?

RTK (Rust Token Killer) adalah CLI proxy yang:
- Memfilter dan mengkompresi output command sebelum sampai ke LLM context window
- Mengurangi token consumption 60–90% pada command umum
- Single Rust binary, overhead < 10ms
- Mendukung 100+ command: git, npm, composer, php artisan, docker, psql, dll

**Cara kerja:**
```
Tanpa RTK:  agent → shell → git status → ~2.000 token raw output → agent
Dengan RTK: agent → RTK → git status → ~200 token filtered output → agent
```

**Sumber:** https://github.com/rtk-ai/rtk

---

## Kapan WAJIB Pakai RTK

Gunakan RTK untuk **semua command terminal** yang menghasilkan output lebih dari 10 baris.
Jangan gunakan RTK untuk built-in tools Kiro: `read_file`, `grep_search`, `list_directory`, `readCode` — tools ini sudah dioptimasi sendiri.

---

## Command Wajib RTK — ATOMS PROJECT

### Git

| Command Asli | Gunakan RTK |
|-------------|-------------|
| `git status` | `rtk git status` |
| `git diff` | `rtk git diff` |
| `git diff --staged` | `rtk git diff --staged` |
| `git log` | `rtk git log -n 10` |
| `git log --oneline` | `rtk git log -n 20` |
| `git show` | `rtk git show` |
| `git grep "pattern"` | `rtk git grep "pattern"` |
| `git add .` | `rtk git add .` |
| `git commit -m "msg"` | `rtk git commit -m "msg"` |
| `git push` | `rtk git push` |
| `git pull` | `rtk git pull` |

### npm / Node.js (frontend_atoms-maintenance & frontend_atoms)

| Command Asli | Gunakan RTK |
|-------------|-------------|
| `npm install` | `rtk npm install` |
| `npm run dev` | `rtk npm run dev` |
| `npm run build` | `rtk npm run build` |
| `npm run lint` | `rtk lint` |
| `npm test` | `rtk test npm test` |
| `npm run preview` | `rtk npm run preview` |
| `npx tsc --noEmit` | `rtk tsc` |

### PHP / Composer / Laravel (backend_atoms-maintenance & backend_atoms)

| Command Asli | Gunakan RTK |
|-------------|-------------|
| `composer install` | `rtk composer install` |
| `composer update` | `rtk composer update` |
| `composer dump-autoload` | `rtk composer dump-autoload` |
| `php artisan serve` | `rtk php artisan serve` |
| `php artisan test` | `rtk test php artisan test` |
| `php artisan migrate` | `rtk php artisan migrate` |
| `php artisan migrate:status` | `rtk php artisan migrate:status` |
| `php artisan migrate:fresh` | `rtk php artisan migrate:fresh` |
| `php artisan route:list` | `rtk php artisan route:list` |
| `php artisan config:clear` | `rtk php artisan config:clear` |
| `php artisan cache:clear` | `rtk php artisan cache:clear` |
| `php artisan optimize:clear` | `rtk php artisan optimize:clear` |
| `php artisan db:seed` | `rtk php artisan db:seed` |
| `php artisan key:generate` | `rtk php artisan key:generate` |
| `php artisan tinker` | Jangan pakai RTK — interaktif |
| `php artisan queue:work` | Jangan pakai RTK — long-running |

### Docker

| Command Asli | Gunakan RTK |
|-------------|-------------|
| `docker ps` | `rtk docker ps` |
| `docker images` | `rtk docker images` |
| `docker logs <container>` | `rtk docker logs <container>` |
| `docker compose ps` | `rtk docker compose ps` |
| `docker compose up` | `rtk docker compose up` |
| `docker compose logs` | `rtk docker compose logs` |

### PostgreSQL

| Command Asli | Gunakan RTK |
|-------------|-------------|
| `psql -c "\dt"` (output besar) | `rtk psql -c "\dt"` |
| `psql -c "SELECT ..."` (output besar) | `rtk psql -c "SELECT ..."` |

---

## Pola Command RTK

```bash
# Git
rtk git status
rtk git diff
rtk git log -n 10

# Test runners — gunakan rtk test untuk generic wrapper
rtk test php artisan test
rtk test npm test

# Build
rtk npm run build
rtk tsc

# Lint
rtk lint

# Laravel
rtk php artisan route:list
rtk php artisan migrate:status
rtk php artisan config:clear

# Composer
rtk composer install

# Docker
rtk docker compose ps
rtk docker compose logs
```

---

## Aturan Fallback

1. **Jika output RTK terlalu ringkas** dan detail dibutuhkan untuk debug, jalankan command non-RTK hanya untuk scope kecil (satu file, satu test case).
2. **Jangan jalankan command verbose mentah** tanpa alasan — selalu coba RTK dulu.
3. **Jangan pakai RTK untuk command interaktif:** `php artisan tinker`, `psql` (interactive shell), `vim`, `nano`, `npm run dev` (watch mode — gunakan `control_pwsh_process` sebagai gantinya).
4. **Jangan pakai RTK untuk long-running processes:** `php artisan serve`, `npm run dev`, `php artisan queue:work` — gunakan `control_pwsh_process` tool untuk background processes.
5. **Jangan expose secret/token** dari `.env` atau log output. Jika command menghasilkan output yang berisi secret, jangan tampilkan ke user.
6. **Jika RTK tidak tersedia** (belum diinstall), jalankan command biasa tapi batasi output dengan `| head -50` atau `--no-ansi`.

---

## Project-Specific RTK Policy — ATOMS PROJECT

### Wajib via RTK
- Semua pemeriksaan status: `rtk git status`, `rtk git diff`
- Semua test run: `rtk test php artisan test`, `rtk test npm test`
- Semua build: `rtk npm run build`, `rtk tsc`
- Semua migrate: `rtk php artisan migrate`, `rtk php artisan migrate:status`
- Semua route list: `rtk php artisan route:list`
- Semua log check: `rtk docker compose logs`, `rtk docker logs`
- Semua dependency install: `rtk composer install`, `rtk npm install`

### Gunakan `control_pwsh_process` (bukan RTK) untuk
- `php artisan serve` — background process
- `npm run dev` — background process (Vite dev server)
- `php artisan queue:work` — background process

### Urutan Start Services (ATOMS PROJECT)
```bash
# 1. atoms-rostering backend (port 8001)
# → gunakan control_pwsh_process
php artisan serve --port=8001   # di atoms-rostering/backend_atoms

# 2. atoms-maintenance backend (port 8000)
# → gunakan control_pwsh_process
php artisan serve               # di atoms-maintenance/backend_atoms-maintenance

# 3. atoms-rostering frontend (port 5174)
# → gunakan control_pwsh_process
npm run dev -- --port 5174      # di atoms-rostering/frontend_atoms

# 4. atoms-maintenance frontend (port 5173)
# → gunakan control_pwsh_process
npm run dev                     # di atoms-maintenance/frontend_atoms-maintenance
```

### Jika Command Gagal
1. Baca ringkasan RTK terlebih dahulu — RTK menyimpan full output di `~/.local/share/rtk/tee/` jika command gagal.
2. Jika ringkasan tidak cukup, baca file tee log yang disimpan RTK.
3. Jangan langsung re-run command verbose tanpa membaca RTK output dulu.

---

## Cek Instalasi RTK

```bash
rtk --version   # Harus menampilkan versi (misal: rtk 0.28.2)
rtk gain        # Menampilkan token savings stats
```

Jika RTK belum terinstall di Windows:
1. Download `rtk-x86_64-pc-windows-msvc.zip` dari https://github.com/rtk-ai/rtk/releases
2. Extract dan tambahkan `rtk.exe` ke PATH
3. Jalankan dari Command Prompt atau PowerShell (jangan double-click)

> **Catatan Windows:** Auto-rewrite hook tidak bekerja di native Windows (cmd/PowerShell).
> RTK tetap berfungsi tapi harus dipanggil eksplisit: `rtk git status` (bukan `git status` otomatis).
> Untuk full hook support, gunakan WSL.
