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
- **Device client (ncclient)** — `pip install nebula-commander` for enroll and run; see [ncclient documentation](/docs/usage/ncclient/)

## Status

### What's implemented already

- Network Creation
- Certificate Generation
- config.yaml Generation
- Basic Firewall Group Rules
- Node Management
- Audit Logging
- Basic Invitation System
- User Management System

### What's coming in v0.2.0

- Stabilize client
- Client UI via Web Interface
- Magic DNS Implementation
- Exit Nodes

## Installation options

- **Development** — Python backend + React frontend; see [Development: Setup](/docs/development/setup/)
- **NixOS** — Import the module and enable `services.nebula-commander`; see [Server Installation: NixOS](/docs/installation/nixos/)
- **Docker** — See [Server Installation: Docker](/docs/installation/docker/)

## Configuration

Environment variables use the `NEBULA_COMMANDER_` prefix. See [Server Configuration](/docs/configuration/) and [Environment](/docs/configuration/environment/) for key options such as `DATABASE_URL`, `CERT_STORE_PATH`, `JWT_SECRET_KEY` (or `JWT_SECRET_FILE`), `OIDC_ISSUER_URL`, `OIDC_CLIENT_ID`, and `DEBUG`.

## License

Backend and frontend: MIT. Client (ncclient): GPLv3 or later. See [Getting started: License](/docs/getting-started/#license) for details.
