# Library Management System

An API-first library catalogue project built incrementally with Python 3.12, FastAPI, SQLAlchemy, SQLite, and React/TypeScript. Its source catalogue is `books_dataset.csv` in this directory. The raw CSV is intentionally ignored by Git because it is approximately 74 MiB; retain your own local copy and do not commit it.

## Implementation status

- Dataset inspected: UTF-8 CSV, 85,849 records, 13 columns, no malformed row widths, no duplicate ISBN strings.
- Strict ISBN-13 validation rejected 292 source identifiers during import; see [Database design](docs/DATABASE_DESIGN.md).
- Architecture and implementation sequence are documented in `docs/`.
- Backend virtual environment uses Python 3.12.10; pinned dependencies installed successfully and `pip check` passed.
- Phase B1 is implemented: FastAPI app, environment-backed settings, versioned router, health endpoint, OpenAPI and environment-based CORS.
- Phase B2 is implemented: SQLAlchemy 2.0 models, database engine/session dependency and SQLite foreign-key enforcement.
- Phase B3 is implemented: Alembic initial migration and tested upgrade/downgrade workflow.
- Phase B4 is implemented: Pydantic book/user response schemas and bounded pagination schemas.
- Phase B5 is implemented: streaming CSV importer, per-batch transactions, idempotent ISBN upserts, dry-run support, and row-level reports.
- Phase B6 is implemented: public book search/detail and authenticated admin catalogue create/update/delete APIs.
- Phase B7 is implemented: searchable author/genre/publisher endpoints, public book reviews and owner-controlled reading lists.
- Phase B8 is implemented: Argon2 password hashing, signed expiring JWTs, registration/login, current user/profile, password changes and role-based dependencies.
- Phase B9 is implemented: JSON by default, negotiated XML responses and 406 for unsupported Accept types.
- Phase B10 is implemented: structured API errors, safer logs, security headers and production settings checks. Rate limiting must be supplied by a shared ingress before public deployment.
- Phase B11 is implemented: 60 backend tests pass and Ruff reports no issues.
- Phase B12 is implemented: a fresh Python 3.12 environment installed, migrated, tested, and smoke-tested with Uvicorn. The full CSV has been imported into the local SQLite database.
- Frontend phases F0–F10 are implemented: responsive routed app, typed API client, in-memory auth, searchable catalogue/details, reviews and reading-list workflows, editable profile, admin catalogue operations, isolated XML demo, and automated tests.
- Frontend verification passes: type-check, Oxlint, all 32 Vitest tests, and production build. Backend verification passes: 60 pytest tests and Ruff.

## Windows PowerShell setup

From `E:\Venky\codebase\python`:

```powershell
py -3.12 --version
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python --version
python -m pip install --upgrade pip
python -m pip install -r backend\requirements.txt
Copy-Item backend\.env.example backend\.env
```

Generate a new local JWT secret and paste it into `JWT_SECRET_KEY` in `backend\.env` before using authenticated endpoints:

```powershell
python -c "import secrets; print(secrets.token_urlsafe(48))"
```

Do not commit `.env` files. Access tokens expire after 30 minutes by default. Logout is stateless and tokens remain valid until expiry.

The frontend keeps bearer tokens in memory only, so refreshing the page signs the user out. UI role checks improve navigation only; the backend independently authorizes every protected request.

Apply database migrations from the project root:

```powershell
python -m alembic -c backend\alembic.ini upgrade head
```

Start the API from the project root:

```powershell
python -m pytest
python -m uvicorn app.main:app --app-dir backend --reload
```

Configure `CORS_ORIGINS=http://localhost:5173` in `backend\.env`. In a second terminal start the frontend:

```powershell
Set-Location frontend
npm install
Copy-Item .env.example .env
npm run dev
```

Open `http://localhost:5173` (the hostname is part of the allowed CORS origin). The frontend's API URL defaults to `http://127.0.0.1:8000/api/v1`. The checked local `library.db` contains the imported catalogue; a fresh local database is empty until the dataset importer is run.

The health endpoint returns `{"status":"ok"}` at `http://127.0.0.1:8000/api/v1/health`. Interactive OpenAPI docs are at `http://127.0.0.1:8000/docs`.

If PowerShell blocks activation, use the interpreter directly:

```powershell
.\.venv\Scripts\python.exe --version
.\.venv\Scripts\python.exe -m pip install -r backend\requirements.txt
```

## Dataset

The importer streams the CSV in batches, validates ISBN-13 values and row fields, parses genre arrays as JSON, and preserves plain-text genres as single labels. Missing optional source values remain `NULL`; no missing data is fabricated. It reports row issues and supports `--dry-run`. A full-dataset dry run using a temporary SQLite database processed 85,849 rows: 85,557 eligible and 292 rejected for ISBN validation; all writes were rolled back.

The full import into the checked local database processed 85,849 rows: 85,557 books were inserted and 292 rows were rejected for invalid ISBN-13 values. The database now contains 85,557 books, 24,507 authors, 557 genres, and 6,348 publishers. The importer can be rerun safely because it upserts books by ISBN.

After applying migrations, run the importer from the project root so the default SQLite URL resolves to the same database used by Alembic:

```powershell
$env:PYTHONPATH = (Resolve-Path backend).Path
python backend\scripts\import_books.py --dry-run
python backend\scripts\import_books.py
```

Pass `--csv E:\path\to\books.csv` to use another source file or `--batch-size 1000` to change the transaction batch size. A dry run reports the same expected counts but rolls back all database writes. See [Project plan](docs/PROJECT_PLAN.md) and [Database design](docs/DATABASE_DESIGN.md).

## Project documentation

- [Backend implementation guide](docs/BACKEND_IMPLEMENTATION_GUIDE.md)
- [Frontend implementation guide](docs/FRONTEND_IMPLEMENTATION_GUIDE.md)
- [Project plan](docs/PROJECT_PLAN.md)
- [Database design](docs/DATABASE_DESIGN.md)
- [API specification](docs/API_SPECIFICATION.md)
- [Future AI/RAG roadmap](docs/AI_RAG_ROADMAP.md)

## Current phase

Backend phases B1–B12 and frontend phases F0–F10 are complete. The separate RAG roadmap remains deferred until the MVP is complete and approved.
