---
title: Docker
linkTitle: Docker (recommended)
weight: 10
---

Run Nebula Commander with Docker using pre-built images or a local build. You can download all required files without cloning the repository. For HTTPS and HSTS at the edge (Nginx, Traefik, or Caddy in front of the frontend container), see [Reverse Proxy](/docs/installation/reverse-proxy/).

## Downloading files

You need: `docker-compose.yml`, `docker-compose-keycloak.yml`, `.env.example`, and the `env.d.example/` directory (backend and Keycloak config). Either use the commands below or the provided script.

### Using curl

Run these in an empty directory (or where you want the Docker setup):

```bash
BASE_URL="https://raw.githubusercontent.com/NixRTR/nebula-commander/main/docker"

curl -sSL -o docker-compose.yml "${BASE_URL}/docker-compose.yml"
curl -sSL -o docker-compose-keycloak.yml "${BASE_URL}/docker-compose-keycloak.yml"
curl -sSL -o .env.example "${BASE_URL}/.env.example"

mkdir -p env.d.example/keycloak
curl -sSL -o env.d.example/backend "${BASE_URL}/env.d.example/backend"
curl -sSL -o env.d.example/keycloak/keycloak "${BASE_URL}/env.d.example/keycloak/keycloak"
curl -sSL -o env.d.example/keycloak/postgresql "${BASE_URL}/env.d.example/keycloak/postgresql"
```

Then create the Docker network (required by the compose files):

```bash
docker network create nebula-commander
```

Copy the example env into place and edit as needed:

```bash
cp .env.example .env
cp -r env.d.example env.d
# Edit env.d/backend (JWT secret, OIDC, etc.)
```

### Using the download script

The repository provides a script that downloads the same files and checks for prerequisites. It does **not** install anything; if Docker or Docker Compose is missing, it prints what you need and exits.

```bash
curl -sSL https://raw.githubusercontent.com/NixRTR/nebula-commander/main/docker/download.sh | bash
```

Then run the steps the script prints: copy `.env.example` to `.env`, copy `env.d.example` to `env.d`, edit `env.d/backend`, create the network, and start with `docker compose up -d`.

---

## Example file contents

Below are the default file contents for reference. After downloading, copy to `.env` and `env.d/` and customize.

### docker-compose.yml

```yaml
name: nebulacdr

include:
  - path: ./docker-compose-keycloak.yml

services:

  backend:
    build:
      context: ..
      dockerfile: docker/backend/Dockerfile
    image: ghcr.io/nixrtr/nebula-commander-backend:latest
    container_name: nebula-commander-backend
    restart: unless-stopped
    ports:
      - "${BACKEND_PORT:-8081}:8081"
    volumes:
      # Persistent data storage
      - nebula-commander-data:/var/lib/nebula-commander
      # Optional: mount JWT secret file
      - ${JWT_SECRET_FILE:-/dev/null}:/run/secrets/jwt-secret:ro
    env_file:
      # Backend configuration (database, JWT, OIDC, CORS, debug)
      - env.d/backend
    environment:
      - NEBULA_COMMANDER_SERVER_HOST=0.0.0.0
      - NEBULA_COMMANDER_SERVER_PORT=8081
    healthcheck:
      test: ["CMD", "python3", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:8081/api/health')"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
    networks:
      - nebula-commander

  frontend:
    build:
      context: ..
      dockerfile: docker/frontend/Dockerfile
    image: ghcr.io/nixrtr/nebula-commander-frontend:latest
    container_name: nebula-commander-frontend
    restart: unless-stopped
    ports:
      - "${FRONTEND_PORT:-80}:80"
    depends_on:
      backend:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 5s
    networks:
      - nebula-commander

networks:
  nebula-commander:
    external: true

volumes:
  nebula-commander-data:
    driver: local
```

### docker-compose-keycloak.yml

```yaml
# Keycloak OIDC Authentication Stack
# To use: docker compose -f docker-compose.yml -f docker-compose-keycloak.yml up -d

services:
  keycloak_db:
    image: postgres:16-alpine
    container_name: keycloak-db
    restart: unless-stopped
    env_file:
      - env.d/keycloak/postgresql
    volumes:
      - keycloak-db-data:/var/lib/postgresql/data
    networks:
      - nebula-commander
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER:-keycloak}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

  keycloak:
    build:
      context: ..
      dockerfile: docker/keycloak/Dockerfile
    image: ghcr.io/nixrtr/nebula-commander-keycloak:latest
    container_name: keycloak
    restart: unless-stopped
    env_file:
      - env.d/keycloak/keycloak
      - env.d/backend
    ports:
      - "${KEYCLOAK_PORT:-8080}:8080"
      - "${KEYCLOAK_ADMIN_PORT:-9000}:9000"
    depends_on:
      keycloak_db:
        condition: service_healthy
    networks:
      - nebula-commander
    healthcheck:
      test: ["CMD-SHELL", "exec 3<>/dev/tcp/127.0.0.1/8080 && echo -e 'GET /health/ready HTTP/1.1\\r\\nhost: 127.0.0.1\\r\\nConnection: close\\r\\n\\r\\n' >&3 && cat <&3 | grep -q '200 OK'"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 60s

volumes:
  keycloak-db-data:
    driver: local

networks:
  nebula-commander:
    external: true
```

### .env.example

```bash
# Nebula Commander Docker Infrastructure Configuration
# Copy to .env and customize. Backend settings go in env.d/backend.

# Port for the frontend (Nginx)
FRONTEND_PORT=80

# Port for the backend (FastAPI)
# BACKEND_PORT=8081

# Port for Keycloak (when using docker-compose-keycloak.yml)
# KEYCLOAK_PORT=8080

# Path to JWT secret file on host (optional)
# JWT_SECRET_FILE=/path/to/jwt-secret.txt
```

### env.d.example/backend

Full example. See [Configuration: Environment](/docs/configuration/environment/) for all options.

```bash
# =============================================================================
# Nebula Commander Backend Configuration
# =============================================================================
# This file contains all backend-specific settings including database, JWT,
# OIDC, CORS, and debug mode. All variables use the NEBULA_COMMANDER_ prefix.

# =============================================================================
# Database Configuration
# =============================================================================

# SQLite database URL: use four slashes for absolute path (so DB is at /var/lib/..., not CWD/var/lib/...)
NEBULA_COMMANDER_DATABASE_URL=sqlite+aiosqlite:////var/lib/nebula-commander/db.sqlite

# Certificate storage path (inside container)
NEBULA_COMMANDER_CERT_STORE_PATH=/var/lib/nebula-commander/certs

# =============================================================================
# JWT Authentication
# =============================================================================

# JWT secret key for signing tokens
# WARNING: Generate a secure random key for production! Use: openssl rand -base64 32
# DO NOT use this example value in production!
NEBULA_COMMANDER_JWT_SECRET_KEY=CHANGE_ME_GENERATE_RANDOM_32_CHARS_MIN

# Alternative: Use a secret file (more secure)
# NEBULA_COMMANDER_JWT_SECRET_FILE=/run/secrets/jwt-secret

# JWT algorithm (HS256, HS384, HS512)
NEBULA_COMMANDER_JWT_ALGORITHM=HS256

# JWT token expiration in minutes (1440 = 24 hours)
NEBULA_COMMANDER_JWT_EXPIRATION_MINUTES=1440

# =============================================================================
# Encryption at rest (required)
# =============================================================================
# Fernet key for encrypting sensitive DB columns and cert store files.
# Generate: python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
# Required at startup - the backend refuses to boot without one.
NEBULA_COMMANDER_ENCRYPTION_KEY=your-fernet-key-here

# Alternative: key from file
# NEBULA_COMMANDER_ENCRYPTION_KEY_FILE=/run/secrets/encryption-key

# =============================================================================
# Public URL (required for OIDC and redirects)
# =============================================================================
# Base URL where users reach the app (browser). All auth redirects and callbacks
# use this. Supports FQDN or host:port. Backend and frontend may be on different hosts.
# Examples: https://nebula.example.com  |  http://192.168.1.1:9091
# If set, OIDC redirect URI is derived as PUBLIC_URL + /api/auth/callback.
NEBULA_COMMANDER_PUBLIC_URL=https://nebula.example.com

# =============================================================================
# Standalone Admin Bootstrap (only relevant if you skip OIDC entirely)
# =============================================================================
# When no OIDC provider is configured below, /api/auth/dev-token is disabled by
# default. If you're running standalone with no IdP and need the UI to be able
# to self-issue an initial admin session, opt in explicitly here.
# WARNING: This grants unauthenticated system-admin access to anyone who can
# reach the backend. Only enable for a genuinely no-IdP deployment.
# NEBULA_COMMANDER_STANDALONE_ADMIN_BOOTSTRAP=true

# =============================================================================
# OIDC Configuration (Optional)
# =============================================================================
# Uncomment and configure to use OIDC authentication with Keycloak or other providers.
# All user-facing auth (login redirect, callback, logout) goes through the public URL above.

# OIDC Issuer URL (internal - backend→Keycloak, same Docker network only)
# Use CONTAINER port (8080). If Keycloak host port is 8082, still use 8080 here.
NEBULA_COMMANDER_OIDC_ISSUER_URL=http://keycloak:8080/realms/nebula-commander

# OIDC Public Issuer URL (external - for browser: logout, discovery)
# Keycloak as seen by the user's browser: FQDN or host:port.
# Examples: https://auth.example.com/realms/nebula-commander  |  http://host:8082/realms/nebula-commander
NEBULA_COMMANDER_OIDC_PUBLIC_ISSUER_URL=https://auth.example.com/realms/nebula-commander

# OIDC Client ID (configured in Keycloak)
NEBULA_COMMANDER_OIDC_CLIENT_ID=nebula-commander

# OIDC Client Secret (from Keycloak client credentials)
# WARNING: Replace with your actual Keycloak client secret! Do not use this placeholder.
NEBULA_COMMANDER_OIDC_CLIENT_SECRET=YOUR_KEYCLOAK_CLIENT_SECRET_HERE

# Alternative: Use a secret file (more secure)
# NEBULA_COMMANDER_OIDC_CLIENT_SECRET_FILE=/run/secrets/oidc-secret

# Redirect URI: derived from NEBULA_COMMANDER_PUBLIC_URL when unset (PUBLIC_URL + /api/auth/callback).
# Override only if you need a different callback URL.
# NEBULA_COMMANDER_OIDC_REDIRECT_URI=

# OIDC Scopes (space-separated)
NEBULA_COMMANDER_OIDC_SCOPES=openid profile email

# =============================================================================
# CORS Configuration
# =============================================================================

# Allowed CORS origins (comma-separated). Include your public app URL (FQDN or host:port).
# SECURITY WARNING: Using '*' with credentials is insecure!
NEBULA_COMMANDER_CORS_ORIGINS=https://nebula.example.com

# =============================================================================
# Session Security
# =============================================================================

# Set session cookie to HTTPS-only (recommended for production)
NEBULA_COMMANDER_SESSION_HTTPS_ONLY=false

# =============================================================================
# Redirect Security
# =============================================================================

# Allowed hosts for OAuth/OIDC redirects (comma-separated)
# When empty and NEBULA_COMMANDER_PUBLIC_URL is set, derived from PUBLIC_URL (recommended).
NEBULA_COMMANDER_ALLOWED_REDIRECT_HOSTS=

# =============================================================================
# Email / SMTP Configuration (Optional)
# =============================================================================

NEBULA_COMMANDER_SMTP_ENABLED=false
NEBULA_COMMANDER_SMTP_HOST=smtp.gmail.com
NEBULA_COMMANDER_SMTP_PORT=587
NEBULA_COMMANDER_SMTP_USE_TLS=true
# For Gmail, use an App Password, not your regular password.
NEBULA_COMMANDER_SMTP_USERNAME=your-email@gmail.com
NEBULA_COMMANDER_SMTP_PASSWORD=your-app-password
# Alternative: NEBULA_COMMANDER_SMTP_PASSWORD_FILE=/run/secrets/smtp-password
NEBULA_COMMANDER_SMTP_FROM_EMAIL=noreply@example.com
NEBULA_COMMANDER_SMTP_FROM_NAME=Nebula Commander

# =============================================================================
# Analytics (Optional)
# =============================================================================
# When set, the frontend fetches /api/public-config and injects the corresponding
# scripts into the page. No auth required; config is public.

# NEBULA_COMMANDER_PLAUSIBLE_DOMAIN=example.com
# NEBULA_COMMANDER_PLAUSIBLE_SCRIPT_SRC=https://plausible.example.com/js/script.file-downloads.hash.outbound-links.js
# NEBULA_COMMANDER_GA_MEASUREMENT_ID=
# NEBULA_COMMANDER_ANALYTICS_CUSTOM_SCRIPTS=[{"src":"https://example.com/script.js","defer":true}]

# =============================================================================
# Debug Mode
# =============================================================================

# Enable debug mode (enables /dev-token endpoint). Never enable in production!
NEBULA_COMMANDER_DEBUG=false
```

### env.d.example/keycloak/keycloak

```bash
# Keycloak Configuration
KC_DB=postgres
KC_DB_URL_HOST=keycloak_db
KC_DB_URL_PORT=5432
KC_DB_URL_DATABASE=keycloak
KC_DB_USERNAME=keycloak
KC_DB_PASSWORD=keycloak_db_password

KC_BOOTSTRAP_ADMIN_USERNAME=admin
KC_BOOTSTRAP_ADMIN_PASSWORD=admin

KC_HOSTNAME=localhost
KC_HOSTNAME_STRICT=false
KC_HTTP_ENABLED=true
KC_HEALTH_ENABLED=true
KC_METRICS_ENABLED=true
KC_LOG_LEVEL=info
```

### env.d.example/keycloak/postgresql

```bash
# PostgreSQL Configuration for Keycloak
POSTGRES_DB=keycloak
POSTGRES_USER=keycloak
POSTGRES_PASSWORD=keycloak_db_password
```

---

## Quick start (pre-built images)

After you have the files and `env.d/backend` configured:

```bash
docker network create nebula-commander   # if not already created
docker compose pull
docker compose up -d
docker compose logs -f
```

The application is available at http://localhost (or the port set in `.env`).

## Building locally

If you prefer to build images instead of pulling:

```bash
docker compose build
docker compose up -d
```

See [Development: Manual builds](/docs/development/manual-builds/#docker) for build-args and multi-arch builds.

## Images

| Image | Registry | Base | Port | Platforms |
|-------|-----------|------|------|-----------|
| Backend | `ghcr.io/nixrtr/nebula-commander-backend:latest` | Python 3.13-slim | 8081 | linux/amd64, linux/arm64 |
| Frontend | `ghcr.io/nixrtr/nebula-commander-frontend:latest` | nginx:alpine | 80 | linux/amd64, linux/arm64 |
| Keycloak | `ghcr.io/nixrtr/nebula-commander-keycloak:latest` | Keycloak | 8080 | linux/amd64, linux/arm64 |

## Architecture

- **Frontend (Nginx)** – Serves the React SPA and proxies `/api/*` to the backend. Port 80.
- **Backend (FastAPI)** – REST API, certificate management, SQLite. Port 8081.
- **Persistent volume** – SQLite database and Nebula certificates under `/var/lib/nebula-commander`.

```
Frontend (Nginx) :80  -->  Backend (FastAPI) :8081  -->  Volume (db + certs)
```

## Configuration

Use two places:

1. **Infrastructure** – `.env`: ports, optional JWT secret file path.
2. **Backend** – `env.d/backend`: all `NEBULA_COMMANDER_*` variables (database, JWT, OIDC, CORS, SMTP, debug).

See [Configuration](/docs/configuration/) for full option lists.

## With Keycloak (OIDC)

The default `docker-compose.yml` **includes** the Keycloak stack via `include: path: ./docker-compose-keycloak.yml`, so a single command starts backend, frontend, and Keycloak:

```bash
docker compose up -d
```

Configure OIDC in `env.d/backend` and optionally use the zero-touch Keycloak setup. Details: [Configuration: OIDC](/docs/configuration/oidc/).

## Without Keycloak

Run only the backend and frontend. The backend exposes `/api/auth/dev-token` when OIDC is not configured (suitable for development only).
