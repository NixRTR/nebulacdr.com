---
title: Getting Started
linkTitle: Getting Started
weight: 10
---

Nebula Commander is a self-hosted control plane for [Nebula](https://github.com/slackhq/nebula) overlay networks. You run the server yourself and manage networks, nodes, certificates, and access from a web UI and optional device client.

## Features

- **Networks and nodes** – Create networks, manage nodes, IP allocation, and certificates
- **Web UI** – React dashboard with OIDC (for example, Keycloak) or a dev token for local development
- **Device client (ncclient)** – Enroll devices with a one-time code, then run as a daemon to pull config and certificates and optionally run or restart Nebula when config changes. Install with `pip install nebula-commander`; see [ncclient](/docs/usage/ncclient/) for details.

## Status

Early development. Core APIs and UI are implemented.

## Prerequisites

- **Nebula** – You need [Nebula](https://github.com/slackhq/nebula) installed on devices that will join your networks. Nebula Commander issues certificates and config; the Nebula binary runs on each node.
- **Server** – A machine or container to run the backend (and optionally the frontend). Python 3.10+ for development; Docker or NixOS for deployment.

## Next steps

1. Choose an [installation method](/docs/installation/) (Docker or NixOS), or [development setup](/docs/development/setup/) for local hacking.
2. [Configure](/docs/configuration/) the backend (database, JWT, and optionally OIDC and SMTP).
3. Use the [Web UI](/docs/web-ui/) and [ncclient](/docs/usage/ncclient/) as needed.

## License

Backend and frontend: MIT. Client (ncclient): GPLv3 or later. See the [repository](https://github.com/NixRTR/nebula-commander) for details.
