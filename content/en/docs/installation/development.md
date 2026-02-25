---
title: Development Setup
linkTitle: Development
weight: 30
---

Run the backend and frontend locally for development.

## Backend

From the `nebula-commander` repository root:

```bash
python -m venv .venv
source .venv/bin/activate   # or .venv\Scripts\activate on Windows
pip install -r backend/requirements.txt
export NEBULA_COMMANDER_DATABASE_URL="sqlite+aiosqlite:///./backend/db.sqlite"
export NEBULA_COMMANDER_CERT_STORE_PATH="./backend/certs"
export DEBUG=true
python -m uvicorn backend.main:app --reload --port 8081
```

Use a real JWT secret in production; for local dev, `DEBUG=true` enables the dev-token endpoint.

## Frontend

In another terminal:

```bash
cd frontend && npm install && npm run dev
```

Open http://localhost:5173. When the backend is in debug mode, you can log in via the dev token (no OIDC required).

## Configuration

Set at least:

- `NEBULA_COMMANDER_DATABASE_URL` – SQLite path (e.g. `sqlite+aiosqlite:///./backend/db.sqlite`)
- `NEBULA_COMMANDER_CERT_STORE_PATH` – Directory for CA and host certs
- `DEBUG=true` – Enables dev token and hot reload

See [Configuration: Environment](/docs/configuration/environment/) for all options.
