# SMART LIB — Phase 2

A full-stack Library Management System being developed with the approved stack:
HTML, CSS, JavaScript, Node.js, Express, SQLite, better-sqlite3, bcryptjs and session-based authentication.

## Requirements

- Node.js 20 or newer
- npm
- VS Code (recommended)
- Google Chrome (for later frontend phases)

## 1. Install dependencies

Open a terminal in the `SMART-LIB` project folder:

```bash
npm install
```

## 2. Configure environment

Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

macOS/Linux:

```bash
cp .env.example .env
```

Open `.env`. Replace `SESSION_SECRET` with a long random value (at least 32 characters). Keep `.env` private; it is excluded by `.gitignore`.

## 3. Initialize the database

```bash
npm run db:init
```

This creates `data/smartlib.sqlite` and the seven approved tables. Initialization is safe to repeat.

## 4. Seed demonstration data

```bash
npm run db:seed
```

This creates the demo accounts, six categories and eight sample books. It is idempotent: running it again will not duplicate seeded records.

### Local demonstration credentials

| Role | Email | Password |
|---|---|---|
| Admin | admin@smartlib.local | Admin@12345 |
| Student | student@smartlib.local | Student@12345 |

These are local demonstration credentials. Change them before any deployment. Passwords are stored as bcrypt hashes, never as plaintext.

## 5. Run database and health tests

```bash
npm test
```

Or database-only tests:

```bash
npm run test:db
```

## 6. Start the server

```bash
npm start
```

Open:

- Application root: http://localhost:3000
- Health endpoint: http://localhost:3000/api/health

Expected health response (timestamp will differ):

```json
{
  "status": "ok",
  "service": "SMART LIB API",
  "database": "connected",
  "timestamp": "2026-11-19T00:00:00.000Z"
}
```

The health endpoint checks an actual SQLite query. It is not just a static response.

For automatic restart during development:

```bash
npm run dev
```

## Database notes

- SQLite foreign-key enforcement is enabled on the application connection.
- WAL journaling and a 5-second busy timeout are configured.
- Tables use foreign keys, `CHECK`, `UNIQUE` constraints and indexes.
- A database trigger blocks permanent deletion of books that have borrowing requests or loan history. Such books must be archived in the future admin module.
- Fine calculations, issuing, returns, authentication and role authorization are intentionally implemented in later phases.

## Troubleshooting

**`better-sqlite3` installation fails:** use Node.js 20 LTS or newer, then run `npm install` again. On Windows, native build tools may be needed if npm cannot download a prebuilt binary.

**Port 3000 is busy:** set another `PORT` value in `.env`, for example `PORT=3001`.

**Database needs a fresh start:** stop the server, back up any data you need, then delete `data/smartlib.sqlite` and run `npm run db:setup`. This permanently removes local database records.

## Current scope

Phase 2 includes project setup, schema, seeding, a health endpoint and automated tests. Login/session behavior and the web interface are not yet implemented.


## Phase 3 — Authentication (implementation prepared)

Authentication endpoints:

- `POST /api/auth/register` — creates student accounts only.
- `POST /api/auth/login` — authenticates credentials; an optional `role` field selects the expected login portal (`admin` or `student`).
- `POST /api/auth/logout` — destroys the current session.
- `GET /api/auth/me` — reports the current authenticated account.

Example student registration body:

```json
{
  "name": "Example Student",
  "email": "student@example.com",
  "password": "StrongPass123"
}
```

Example admin login body:

```json
{
  "email": "admin@smartlib.local",
  "password": "Admin@12345",
  "role": "admin"
}
```

Example student login body:

```json
{
  "email": "student@smartlib.local",
  "password": "Student@12345",
  "role": "student"
}
```

Authentication uses the existing `better-sqlite3` connection for Express session storage. No second SQLite driver is introduced. Run `npm test` to execute the full test suite. Phase 3 is not complete until those tests pass locally.
