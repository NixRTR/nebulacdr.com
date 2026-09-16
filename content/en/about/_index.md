---
title: About Nebula Commander
linkTitle: Status
menu:
  main:
    weight: 10
---

Nebula Commander is a **self-hosted control plane** for [Nebula](https://github.com/slackhq/nebula) overlay networks.

## What it does

- **Networks & nodes** — Create networks, manage nodes, IP allocation, and certificates
- **Web UI** — React dashboard with OIDC (e.g. Keycloak) or dev token authentication
- **Device client (ncclient)** — `pip install nebula-commander` for enroll and run; see [ncclient documentation](/docs/usage/ncclient/)

## Status (as of v0.4.0)

### What's implemented already

- Networks, nodes, IP allocation, and certificate management — including automatic
  re-signing when a node's group changes, no re-enrollment needed
- Nebula v2 certificates, with P256 as an opt-in curve
- Firewall group rules, with a visual group access diagram
- Magic DNS: split-horizon DNS via dnsmasq (Linux/Docker) and NRPT (Windows), with
  wildcard alias support
- Client UI via web interface — a full React dashboard for networks, nodes, groups,
  DNS, users, invitations, and the audit log, with a consistent square-card layout
  across the list pages
- **Subnet routers and exit nodes** (Nebula's `unsafe_routes`), configurable from
  both sides: the gateway that advertises a route, and a simple "Use Subnet
  Router"/"Use Exit Node" picker on any other node that wants to consume one — see
  [Subnet Routers and Exit Nodes](/docs/usage/unsafe-routes/)
- **Per-account color theming** — every user can customize button, status, badge,
  and background colors independently for light and dark mode, and save named
  presets to switch between — see [Appearance](/docs/web-ui/appearance/)
- Device client (`ncclient`): CLI, a Windows service + tray app, a Docker image, and
  a NixOS module (`services.ncclient`), plus mobile support (iOS/Android via the
  official Mobile Nebula app)
- Lighthouse-based peer reachability monitoring and node offline detection
- OIDC (e.g. Keycloak) or dev-token authentication, with step-up reauth required for
  sensitive actions (deletions, revocations)
- Audit logging, an invitation system, and multi-user management
- Encryption at rest for certificates and keys

### What's still planned

- [ ] Automatic host-level route/NAT configuration for subnet routers and exit
  nodes on Windows and Docker-deployed nodes (Linux nodes running `ncclient`
  already get this automatically; see
  [Subnet Routers and Exit Nodes](/docs/usage/unsafe-routes/))
- [ ] Continued client hardening as real-world deployments surface edge cases

## Installation options

- **Development** — Python backend + React frontend; see [Development: Setup](/docs/development/setup/)
- **NixOS** — Import the module and enable `services.nebula-commander`; see [Server Installation: NixOS](/docs/installation/nixos/)
- **Docker** — See [Server Installation: Docker](/docs/installation/docker/)

## Configuration

Environment variables use the `NEBULA_COMMANDER_` prefix. See [Server Configuration](/docs/configuration/) and [Environment](/docs/configuration/environment/) for key options such as `DATABASE_URL`, `CERT_STORE_PATH`, `JWT_SECRET_KEY` (or `JWT_SECRET_FILE`), `OIDC_ISSUER_URL`, `OIDC_CLIENT_ID`, and `DEBUG`.

## License

Backend and frontend: MIT. Client (ncclient): GPLv3 or later. See [Getting started: License](/docs/getting-started/#license) for details.
