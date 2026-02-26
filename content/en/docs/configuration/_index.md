---
title: Server Configuration
linkTitle: Server Configuration
weight: 30
---

Nebula Commander is configured with environment variables. All backend settings use the prefix `NEBULA_COMMANDER_`. You can configure everything from the docs below.

- **[Environment variables](/docs/configuration/environment/)** – Full list: database, JWT, OIDC, CORS, session, SMTP, debug. Use this to configure the backend completely.
- **[OIDC](/docs/configuration/oidc/)** – Log in with Keycloak or another OIDC provider: issuer URLs (internal vs public), client id/secret, redirect URI, and zero-touch vs manual setup.

## Two-tier layout (Docker)

When using Docker:

- **Infrastructure** – `docker/.env`: ports (e.g. frontend, backend, Keycloak), optional `JWT_SECRET_FILE` path.
- **Backend** – `docker/env.d/backend`: every `NEBULA_COMMANDER_*` variable. Copy from `env.d.example/backend` and edit.

See [Installation: Docker](/docs/installation/docker/#configuration) for the copy steps.

## NixOS

NixOS options (port, backendPort, databasePath, certStorePath, jwtSecretFile, debug) are documented in [Installation: NixOS](/docs/installation/nixos/). For OIDC and other backend-only settings, extend the service environment or point the backend at a config file if your setup supports it.
