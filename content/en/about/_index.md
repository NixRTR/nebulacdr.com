---
title: About Nebula Commander
linkTitle: About
menu:
  main:
    weight: 10
---

Nebula Commander is a **self-hosted control plane** for [Nebula](https://github.com/slackhq/nebula) overlay networks.

## What it does

- **Networks & nodes** — Create networks, manage nodes, IP allocation, and certificates
- **Web UI** — React dashboard with OIDC (e.g. Keycloak) or dev token authentication
- **Device client (ncclient)** — `pip install nebula-commander` for enroll and run; see the [client README](https://github.com/NixRTR/nebula-commander/blob/main/client/README.md)

## Status

Early development. Core APIs and UI are implemented.

## Installation options

- **Development** — Python backend + React frontend; see the [main README](https://github.com/NixRTR/nebula-commander#installation)
- **NixOS** — Import the module and enable `services.nebula-commander`
- **Docker** — See [docker/README.md](https://github.com/NixRTR/nebula-commander/tree/main/docker) in the repo

## Configuration

Environment variables use the `NEBULA_COMMANDER_` prefix. Key options include `DATABASE_URL`, `CERT_STORE_PATH`, `JWT_SECRET_KEY` (or `JWT_SECRET_FILE`), `OIDC_ISSUER_URL`, `OIDC_CLIENT_ID`, and `DEBUG`.

## License

Backend and frontend: MIT. Client (ncclient): GPLv3 or later. See [LICENSE.md](https://github.com/NixRTR/nebula-commander/blob/main/LICENSE.md) in the repository.
