---
title: Environment Variables
linkTitle: Environment
weight: 10
---

All backend settings use the `NEBULA_COMMANDER_` prefix. Set them in the environment or in a file (e.g. `docker/env.d/backend`). Optional values can be omitted; defaults apply.

## Application

| Variable | Description | Default |
|----------|-------------|---------|
| `NEBULA_COMMANDER_DEBUG` | Enable debug mode (enables dev-token endpoint; do not use in production) | `false` |

## Database

| Variable | Description | Default |
|----------|-------------|---------|
| `NEBULA_COMMANDER_DATABASE_URL` | Database URL (SQLite: use four slashes for absolute path, e.g. `sqlite+aiosqlite:////var/lib/nebula-commander/db.sqlite`) | `sqlite+aiosqlite:////var/lib/nebula-commander/db.sqlite` |
| `NEBULA_COMMANDER_DATABASE_PATH` | Override for SQLite path | — |
| `NEBULA_COMMANDER_CERT_STORE_PATH` | Directory for CA and host certificates | `/var/lib/nebula-commander/certs` |

## JWT

| Variable | Description | Default |
|----------|-------------|---------|
| `NEBULA_COMMANDER_JWT_SECRET_KEY` | Secret for signing tokens (generate with e.g. `openssl rand -base64 32`) | `change-this-in-production` |
| `NEBULA_COMMANDER_JWT_SECRET_FILE` | Path to file containing JWT secret (overrides secret key when present) | — |
| `NEBULA_COMMANDER_JWT_ALGORITHM` | JWT algorithm | `HS256` |
| `NEBULA_COMMANDER_JWT_EXPIRATION_MINUTES` | Token expiration in minutes | `1440` (24 hours) |

## Public URL and OIDC

| Variable | Description | Default |
|----------|-------------|---------|
| `NEBULA_COMMANDER_PUBLIC_URL` | Base URL where users reach the app (FQDN or host:port). Used to derive redirect URI and for redirect validation. | — |
| `NEBULA_COMMANDER_OIDC_ISSUER_URL` | OIDC issuer URL used by the backend to reach the provider (internal; e.g. `http://keycloak:8080/realms/nebula-commander`) | — |
| `NEBULA_COMMANDER_OIDC_PUBLIC_ISSUER_URL` | OIDC issuer URL as seen by the browser (FQDN or host:port) | — |
| `NEBULA_COMMANDER_OIDC_CLIENT_ID` | OIDC client ID | — |
| `NEBULA_COMMANDER_OIDC_CLIENT_SECRET` | OIDC client secret | — |
| `NEBULA_COMMANDER_OIDC_CLIENT_SECRET_FILE` | Path to file containing OIDC client secret | — |
| `NEBULA_COMMANDER_OIDC_REDIRECT_URI` | Callback URL (optional; derived as PUBLIC_URL + `/api/auth/callback` when PUBLIC_URL is set) | — |
| `NEBULA_COMMANDER_OIDC_SCOPES` | OIDC scopes (space-separated) | `openid profile email` |

## CORS and session

| Variable | Description | Default |
|----------|-------------|---------|
| `NEBULA_COMMANDER_CORS_ORIGINS` | Allowed CORS origins: `*` or comma-separated list. Include your public app URL. Using `*` with credentials is insecure. | `http://localhost:3000`, `http://localhost:5173` |
| `NEBULA_COMMANDER_SESSION_HTTPS_ONLY` | Set session cookie to HTTPS-only (use true in production with HTTPS) | `false` |
| `NEBULA_COMMANDER_ALLOWED_REDIRECT_HOSTS` | Allowed hosts for OAuth/OIDC redirects (comma-separated). When empty and PUBLIC_URL is set, derived from PUBLIC_URL. | — |

## Certificates and device tokens

| Variable | Description | Default |
|----------|-------------|---------|
| `NEBULA_COMMANDER_DEFAULT_CERT_EXPIRY_DAYS` | Default certificate expiry in days | `365` |
| `NEBULA_COMMANDER_DEVICE_TOKEN_EXPIRATION_DAYS` | Device token (enrollment) expiry in days | `3650` |

## SMTP (optional)

| Variable | Description | Default |
|----------|-------------|---------|
| `NEBULA_COMMANDER_SMTP_ENABLED` | Enable sending email (e.g. for invitations) | `false` |
| `NEBULA_COMMANDER_SMTP_HOST` | SMTP host | `localhost` |
| `NEBULA_COMMANDER_SMTP_PORT` | SMTP port | `587` |
| `NEBULA_COMMANDER_SMTP_USERNAME` | SMTP username | — |
| `NEBULA_COMMANDER_SMTP_PASSWORD` | SMTP password | — |
| `NEBULA_COMMANDER_SMTP_PASSWORD_FILE` | Path to file containing SMTP password | — |
| `NEBULA_COMMANDER_SMTP_USE_TLS` | Use TLS | `true` |
| `NEBULA_COMMANDER_SMTP_FROM_EMAIL` | From address | `noreply@example.com` |
| `NEBULA_COMMANDER_SMTP_FROM_NAME` | From name | `Nebula Commander` |

## Server (advanced)

| Variable | Description | Default |
|----------|-------------|---------|
| `NEBULA_COMMANDER_HOST` | Bind host | `0.0.0.0` |
| `NEBULA_COMMANDER_PORT` | Bind port | `8081` |

## Security notes

- Generate a strong JWT secret for production (e.g. `openssl rand -base64 32`). Do not use the default.
- Prefer `*_FILE` options (JWT, OIDC secret, SMTP password) over plain env vars when possible.
- In production: set `DEBUG=false`, use HTTPS for PUBLIC_URL and OIDC, and set CORS_ORIGINS to your actual frontend origin(s).
