---
title: "State of Nebula Commander Development (February 2025)"
linkTitle: "State of Development (Feb 2025)"
date: 2025-02-24
draft: false
description: "A snapshot of what's implemented, what's in progress, and how you can run or contribute to Nebula Commander."
---

Nebula Commander is a self-hosted control plane for [Nebula](https://github.com/slackhq/nebula) overlay networks. This post summarizes where the project stands as of early 2025 and how you can try it or contribute.

## What’s in place

**Backend (Python/FastAPI)**  
The API supports the core workflows: networks (create, delete), nodes (CRUD, IP allocation, certificates), groups and inbound firewall rules, and OIDC-based access with roles (system-admin, network-owner, user). You can run it with Docker, on NixOS via the systemd module, or locally with a venv and uvicorn.

**Web UI (React)**  
The dashboard lets you manage networks, groups, and nodes; create or sign certificates; generate enrollment codes for the device client; and download ncclient binaries from your own server. Log in via OIDC (e.g. Keycloak) or, in development, with a dev token when `DEBUG=true`.

**Device client (ncclient)**  
The experimental client enrolls with a one-time code, then polls the server for config and certificates and can run or restart Nebula when config changes. Available as a CLI and a Windows tray app; installable via `pip install nebula-commander` or from our [releases](https://github.com/NixRTR/nebula-commander/releases).

**Deployment**  
We document [Docker](https://nebulacdr.com/docs/installation/docker/) (recommended) and [NixOS](https://nebulacdr.com/docs/installation/nixos/) installation, plus [environment and OIDC configuration](https://nebulacdr.com/docs/configuration/). The docs site you’re reading is the project’s main documentation.

## Current status

The project is in **early development**. Core APIs and UI are implemented and usable for running your own Nebula networks with a central server. ncclient is still evolving (CLI and Windows tray; more platforms and packaging may follow). For production-like setups, we recommend using the Web UI and API to manage networks and nodes, then deploying config and certs manually and running Nebula yourself until ncclient stabilizes.

## How to get involved

- **Run it** – Follow [Getting started](https://nebulacdr.com/docs/getting-started/) and [Server Installation](https://nebulacdr.com/docs/installation/), then use the [Web UI](https://nebulacdr.com/docs/web-ui/) to create a network and nodes.
- **Contribute** – Code and issues live on [GitHub](https://github.com/NixRTR/nebula-commander). See [Development](https://nebulacdr.com/docs/development/) for local setup, GitHub Actions, manual builds, and API overview.
- **Stay updated** – Check the [blog](/blog/) and the repo for announcements. We’ll post when we hit milestones (e.g. ncclient stability, new features, or release tagging).

Thanks for your interest in Nebula Commander.
