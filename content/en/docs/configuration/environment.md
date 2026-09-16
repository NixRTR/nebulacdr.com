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
| `NEBULA_COMMANDER_UPDATE_CHECK_ENABLED` | Check GitHub for a newer release, shown as a notification in the About screen. Set to `false` to disable all outbound calls to github.com (air-gapped/privacy-conscious deployments). | `true` |

## Database

| Variable | Description | Default |
|----------|-------------|---------|
| `NEBULA_COMMANDER_DATABASE_URL` | Database URL (SQLite: use four slashes for absolute path, e.g. `sqlite+aiosqlite:////var/lib/nebula-commander/db.sqlite`) | `sqlite+aiosqlite:////var/lib/nebula-commander/db.sqlite` |
| `NEBULA_COMMANDER_DATABASE_PATH` | Override for SQLite path | — |
| `NEBULA_COMMANDER_CERT_STORE_PATH` | Directory for CA and host certificates | `/var/lib/nebula-commander/certs` |

## Encryption at rest

**Required.** The backend refuses to boot if this is left at a placeholder/unset
value — unlike most settings here, there's no insecure-but-working default.

| Variable | Description | Default |
|----------|-------------|---------|
| `NEBULA_COMMANDER_ENCRYPTION_KEY` | Fernet key used to encrypt certificates and private keys at rest. Generate one with `python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"`. | — (required) |
| `NEBULA_COMMANDER_ENCRYPTION_KEY_FILE` | Path to a file containing the Fernet key (overrides the plain env var when present). On NixOS this is auto-generated into a `oneshot` service the first time the module runs. | — |

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
| `NEBULA_COMMANDER_STANDALONE_ADMIN_BOOTSTRAP` | Allow the unauthenticated `/api/auth/dev-token` admin-bootstrap endpoint when no OIDC provider is configured below. Standalone (no-IdP) deployments must opt in explicitly — this grants unauthenticated system-admin access to anyone who can reach the backend, so only enable it for a genuinely no-IdP deployment. Deployments with OIDC configured are unaffected (always 404). | `false` |
| `NEBULA_COMMANDER_OIDC_ISSUER_URL` | OIDC issuer URL used by the backend to reach the provider (internal; e.g. `http://keycloak:8080/realms/nebula-commander`) | — |
| `NEBULA_COMMANDER_OIDC_PUBLIC_ISSUER_URL` | OIDC issuer URL as seen by the browser (FQDN or host:port) | — |
| `NEBULA_COMMANDER_OIDC_CLIENT_ID` | OIDC client ID | — |
| `NEBULA_COMMANDER_OIDC_CLIENT_SECRET` | OIDC client secret | — |
| `NEBULA_COMMANDER_OIDC_CLIENT_SECRET_FILE` | Path to file containing OIDC client secret | — |
| `NEBULA_COMMANDER_OIDC_REDIRECT_URI` | Callback URL (optional; derived as PUBLIC_URL + `/api/auth/callback` when PUBLIC_URL is set) | — |
| `NEBULA_COMMANDER_OIDC_SCOPES` | OIDC scopes (space-separated) | `openid profile email` |
| `NEBULA_COMMANDER_OIDC_ADMIN_ROLE_CLAIM` | Top-level claim name used to detect the system-admin role (e.g. `roles`, `groups`, or an Auth0-style namespaced claim). Leave unset to use Keycloak's default `resource_access.<client_id>.roles` shape. | — |
| `NEBULA_COMMANDER_OIDC_ADMIN_ROLE_VALUE` | Value that must appear in the admin-role claim to grant system-admin | `system-admin` |

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
| `NEBULA_COMMANDER_DEVICE_TOKEN_EXPIRATION_DAYS` | Device token (enrollment) expiry in days | `365` |

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

## Analytics (optional)

Exposed publicly via `GET /api/public-config` (no auth) so the frontend can inject the corresponding script tags.

| Variable | Description | Default |
|----------|-------------|---------|
| `NEBULA_COMMANDER_PLAUSIBLE_DOMAIN` | Plausible domain (e.g. `example.com`); set to enable Plausible analytics | — |
| `NEBULA_COMMANDER_PLAUSIBLE_SCRIPT_SRC` | Custom Plausible script URL (e.g. behind your own proxy), overriding the default script source | — |
| `NEBULA_COMMANDER_GA_MEASUREMENT_ID` | Google Analytics measurement ID (e.g. `G-XXXXXXXXXX`); set to enable GA | — |
| `NEBULA_COMMANDER_ANALYTICS_CUSTOM_SCRIPTS` | JSON array of custom scripts to inject, each `{"src": "https://...", "defer": true}` or `{"inline": "..."}` | — |

## Server (advanced)

| Variable | Description | Default |
|----------|-------------|---------|
| `NEBULA_COMMANDER_HOST` | Bind host | `0.0.0.0` |
| `NEBULA_COMMANDER_PORT` | Bind port | `8081` |

## Security notes

- Generate a strong JWT secret for production (e.g. `openssl rand -base64 32`). Do not use the default.
- `ENCRYPTION_KEY` has no default to leave-unset-and-forget — the backend won't start without one. Generate it once and keep it: losing it makes every stored certificate/private key unrecoverable.
- Prefer `*_FILE` options (JWT, encryption key, OIDC secret, SMTP password) over plain env vars when possible.
- In production: set `DEBUG=false`, use HTTPS for PUBLIC_URL and OIDC, and set CORS_ORIGINS to your actual frontend origin(s). For examples of putting Nebula Commander behind Nginx, Traefik, or Caddy with TLS and HSTS, see [Reverse Proxy](/docs/installation/reverse-proxy/).
