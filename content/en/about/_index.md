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

## Status (as of v0.7.0)

### What's implemented already

- Networks, nodes, IP allocation, and certificate management — including automatic
  re-signing when a node's group changes, no re-enrollment needed
- Nebula v2 certificates, with P256 as an opt-in curve
- Firewall group rules, with a visual group access diagram
- Magic DNS: split-horizon DNS via dnsmasq (Linux/Docker) and NRPT (Windows), with
  wildcard alias support and automatic detection of the host's DNS manager on Linux
- Client UI via web interface — a full React dashboard for networks, nodes, groups,
  DNS, users, invitations, and the audit log, with a consistent square-card layout
  across the list pages
- **Subnet routers and exit nodes** (Nebula's `unsafe_routes`), configurable from
  both sides: the gateway that advertises a route, and a simple "Use Subnet
  Router"/"Use Exit Node" picker on any other node that wants to consume one. The
  gateway's "Used by" list takes whole groups or individual hosts — see
  [Subnet Routers and Exit Nodes](/docs/usage/unsafe-routes/)
- **Public endpoint on any node**, not just lighthouses and relays, added to every
  peer's `static_host_map` so nodes with a known address connect directly, plus
  **additional reachable addresses** (port forwards, second uplinks) a node reports
  to its lighthouses — see
  [Public Endpoints & Reachable Addresses](/docs/usage/reachable-addresses/)
- **Per-account color theming** — every user can customize button, status, badge,
  and background colors independently for light and dark mode, and save named
  presets to switch between — see [Appearance](/docs/web-ui/appearance/)
- Device client (`ncclient`): CLI, native desktop apps for Windows (WinUI 3, with a
  background service) and Linux (GTK4, as `.deb`/`.rpm`/Flatpak), a Docker image,
  a signed apt/rpm package repository (amd64 and arm64), and NixOS modules (`services.ncclient`, `services.ncclient-desktop`), plus mobile support
  (iOS/Android via the official Mobile Nebula app). Only administrators can change a
  device's network settings: an elevated administrator on Windows, the `sudo`/`wheel`
  group on Linux
- **Revocation that holds**: revoking, deleting or re-enrolling a node blocklists its
  old certificate on every other node, and nodes keep running from their last config
  if the server is unreachable — see [Nodes](/docs/web-ui/nodes/#revoke-re-enroll-and-delete)
- **Opt-in automatic client updates** for Windows and Linux packages (notify-only on
  NixOS), turned on by an administrator on the device — see
  [Automatic updates](/docs/usage/ncclient/usage/auto-update/)
- Lighthouse-based peer reachability monitoring and node offline detection
- OIDC (e.g. Keycloak) or dev-token authentication, with step-up reauth required for
  sensitive actions (deletions, revocations)
- Audit logging, an invitation system, and multi-user management
- Encryption at rest for certificates and keys
- **Backup & export** — download the whole instance as one passphrase-encrypted
  file (standard [age](https://age-encryption.org) format) and import it into a
  fresh instance; devices keep working after a move without re-enrolling — see
  [Backup & export](/docs/web-ui/backup/)
- **Hosted option** — [Nebula Commander Cloud](/docs/cloud/) if you'd rather not run
  the server yourself

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
