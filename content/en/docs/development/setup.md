---
title: Development Setup
linkTitle: Setup
weight: 10
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
# Required — the backend refuses to boot without it. Generate one:
# python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
export NEBULA_COMMANDER_ENCRYPTION_KEY="your-fernet-key-here"
export NEBULA_COMMANDER_DEBUG=true
python -m uvicorn backend.main:app --reload --port 8081
```

Use a real JWT secret in production; for local dev, `NEBULA_COMMANDER_DEBUG=true`
enables the dev-token endpoint. Every backend setting uses the
`NEBULA_COMMANDER_` prefix — a bare `DEBUG=true` without it is silently ignored.

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
- `NEBULA_COMMANDER_ENCRYPTION_KEY` – Required; the backend won't start without it
- `NEBULA_COMMANDER_DEBUG=true` – Enables dev token and hot reload

See [Configuration: Environment](/docs/configuration/environment/) for all options.
