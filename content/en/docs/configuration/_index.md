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

`services.nebula-commander` exposes most of these settings as first-class options, including OIDC, `publicUrl`, `corsOrigins`, and `sessionHttpsOnly` — see [Installation: NixOS](/docs/installation/nixos/) for the full option table. Anything not yet exposed as an option can still be set by extending the service `environment` directly with the matching `NEBULA_COMMANDER_*` variable. A separate `services.ncclient` module is also available for running the device client declaratively.
