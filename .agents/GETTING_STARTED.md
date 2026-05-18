# Getting Started

This guide runs the active ATOMS Maintenance project locally:

- Backend: `atoms-maintenance/backend_atoms-maintenance`
- Frontend: `atoms-maintenance/frontend_atoms-maintenance`

Do not modify or run writes against `atoms-rostering`; it is a read-only reference/source-of-truth project.

## Scan Summary

### Package Managers

- Backend uses Composer: `composer.json` and `composer.lock` are present.
- Backend also has a small `package.json` for Laravel/Vite assets, but the API setup is Composer-first.
- Frontend uses npm: `package.json` and `package-lock.json` are present.
- No `yarn.lock` or `pnpm-lock.yaml` was found in the active frontend/backend folders.

### Environment Example Files

- Backend has `.env.example`.
- Frontend does not currently have `.env.example`; create `frontend_atoms-maintenance/.env` manually using the Vite variables below.

### Ports

- Backend: `php artisan serve` defaults to `http://127.0.0.1:8000`.
- Frontend: Vite config does not set a custom port, so `npm run dev` defaults to `http://localhost:5173`.

## Backend (Laravel API)

### Prerequisites

- PHP `^8.3` (from backend `composer.json`)
- Composer
- PostgreSQL
- PHP PostgreSQL extension enabled (`pdo_pgsql`)

### Install Dependencies

```bash
cd atoms-maintenance/backend_atoms-maintenance
composer install
```

### Environment Setup

Create `.env` from the backend example:

```bash
copy .env.example .env
```

Required variables from `backend_atoms-maintenance/.env.example`:

```env
APP_NAME=ATOMS-Maintenance
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=pgsql
DB_HOST=localhost
DB_PORT=5432
DB_DATABASE=atoms_maintenance
DB_USERNAME=postgres
DB_PASSWORD=1234

DEV_MOCK_AUTH=true

SANCTUM_STATEFUL_DOMAINS=localhost:5173
SESSION_DOMAIN=localhost

LOG_CHANNEL=stack
LOG_LEVEL=debug

FILESYSTEM_DISK=local
```

Before migrating, make sure PostgreSQL is running and the database exists:

```sql
CREATE DATABASE atoms_maintenance;
```

Adjust `DB_USERNAME` and `DB_PASSWORD` to match your local PostgreSQL account.

### Generate App Key

```bash
php artisan key:generate
```

### Run Migrations

```bash
php artisan migrate
```

### Seed Development Data

```bash
php artisan db:seed
```

For Work Order demo data specifically:

```bash
php artisan db:seed --class=WorkOrderSeeder
```

### Start Backend Server

```bash
php artisan serve
```

Expected URL:

```text
http://127.0.0.1:8000
```

### Useful Artisan Commands Available

The backend exposes the standard Laravel command set plus Sanctum and project setup commands. Commonly useful commands include:

```bash
php artisan list
php artisan about
php artisan serve
php artisan route:list
php artisan migrate
php artisan migrate:status
php artisan migrate:fresh
php artisan db:seed
php artisan db:show
php artisan db:table
php artisan test
php artisan tinker
php artisan config:clear
php artisan config:cache
php artisan cache:clear
php artisan optimize:clear
php artisan queue:listen
php artisan queue:work
php artisan pail
php artisan storage:link
php artisan sanctum:prune-expired
```

Generator commands are also available, including `make:model`, `make:controller`, `make:migration`, `make:request`, `make:seeder`, `make:policy`, `make:middleware`, `make:test`, and `make:trait`.

## Frontend (React TypeScript + Vite)

### Prerequisites

- Node.js `^20.19.0` or `>=22.12.0` (from installed Vite 8 engine requirement)
- npm

### Install Dependencies

```bash
cd atoms-maintenance/frontend_atoms-maintenance
npm install
```

### Environment Setup

There is no frontend `.env.example` at the moment. Create `frontend_atoms-maintenance/.env` with:

```env
VITE_API_URL=http://localhost:8000/api
VITE_DEV_MOCK_AUTH=true
VITE_ROSTERING_FRONTEND_URL=http://localhost:5174
VITE_REVERB_APP_KEY=atoms-maintenance-key
VITE_REVERB_HOST=localhost
VITE_REVERB_PORT=8080
```

> `VITE_ROSTERING_FRONTEND_URL` digunakan untuk SSO redirect saat `VITE_DEV_MOCK_AUTH=false`.
> Saat mock mode aktif (`true`), variable ini tidak dipakai.

### Start Frontend Dev Server

```bash
npm run dev
```

Expected URL:

```text
http://localhost:5173
```

Vite config file:

```text
atoms-maintenance/frontend_atoms-maintenance/vite.config.ts
```

The config defines the React plugin and `@` alias, but does not override the dev server port.

## Running Order

### Mode 1: Mock Auth (Development — Default)

Cukup jalankan 2 service:

1. Start PostgreSQL.
2. Start backend:
   ```bash
   cd atoms-maintenance/backend_atoms-maintenance
   php artisan serve
   ```
3. Start frontend:
   ```bash
   cd atoms-maintenance/frontend_atoms-maintenance
   npm run dev
   ```
4. Buka `http://localhost:5173/login` — login dengan mock user (password apapun).

### Mode 2: SSO Real (Test Production Flow)

Jalankan 4 service dalam urutan ini:

1. Start PostgreSQL.
2. Start atoms-rostering backend (port 8001):
   ```bash
   cd atoms-rostering/backend_atoms
   php artisan serve --port=8001
   ```
3. Start atoms-maintenance backend (port 8000):
   ```bash
   cd atoms-maintenance/backend_atoms-maintenance
   php artisan serve
   ```
4. Start atoms-rostering frontend (port 5174 — harus eksplisit):
   ```bash
   cd atoms-rostering/frontend_atoms
   npm run dev -- --port 5174
   ```
5. Start atoms-maintenance frontend (port 5173 — default):
   ```bash
   cd atoms-maintenance/frontend_atoms-maintenance
   npm run dev
   ```
6. Set `DEV_MOCK_AUTH=false` di `backend_atoms-maintenance/.env`
7. Set `VITE_DEV_MOCK_AUTH=false` di `frontend_atoms-maintenance/.env`
8. Buka `http://localhost:5174` → login → klik Maintenance → SSO redirect ke `http://localhost:5173`.

## Verify Everything Is Running

### Backend

Open or request:

```text
http://127.0.0.1:8000/api/v1/auth/me
```

With mock auth enabled, API calls must include a mock bearer token such as:

```text
Authorization: Bearer mock-token-1
```

You can also confirm routes with:

```bash
php artisan route:list
```

### Frontend

Open:

```text
http://localhost:5173
```

Expected behavior:

- The app loads in the browser.
- In mock-auth mode, the frontend can call the backend using `VITE_API_URL`.
- Work Order pages should be able to load data from `/api/v1/work-orders` when backend seed data exists.

## Common Errors

### 1. `SQLSTATE[08006]` or database connection refused

Cause: PostgreSQL is not running, the database does not exist, or `.env` credentials are wrong.

Fix:

- Start PostgreSQL.
- Create `atoms_maintenance`.
- Verify `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, and `DB_PASSWORD`.
- Run `php artisan config:clear`.

### 2. `No application encryption key has been specified`

Cause: `APP_KEY` is empty.

Fix:

```bash
php artisan key:generate
```

### 3. Frontend calls fail with CORS, 401, or network errors

Cause: backend is not running, `VITE_API_URL` is wrong, or mock auth settings do not match.

Fix:

- Start backend on port `8000`.
- Set `VITE_API_URL=http://localhost:8000/api`.
- Set backend `DEV_MOCK_AUTH=true`.
- Set frontend `VITE_DEV_MOCK_AUTH=true`.
- Restart the frontend dev server after changing `.env`.

### 4. `npm run dev` fails because Node is unsupported

Cause: Vite 8 requires Node `^20.19.0` or `>=22.12.0`.

Fix:

- Install a supported Node version.
- Re-run `npm install`.

### 5. PHP warning about `pdo_firebird`

Cause: local PHP config references a missing Firebird extension. This project uses PostgreSQL, not Firebird.

Fix:

- Disable or remove the `pdo_firebird` extension entry from your local `php.ini`.
- Ensure `pdo_pgsql` is enabled instead.

