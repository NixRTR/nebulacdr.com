---
title: Installation
linkTitle: Installation
weight: 20
---

You can run Nebula Commander in several ways:

| Method | Best for |
|--------|----------|
| [Docker (recommended)](/docs/installation/docker/) | Quick start and production: pre-built images, `docker compose` |
| [NixOS](/docs/installation/nixos/) | NixOS hosts: systemd module with options for port, database, certs, JWT |
| [Development](/docs/installation/development/) | Local hacking: venv, uvicorn, and frontend dev server |

Choose one and follow the linked page. After installation, see [Configuration](/docs/configuration/) to set the database, JWT, and optional OIDC.
