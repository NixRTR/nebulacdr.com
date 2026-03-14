---
title: Client Usage
linkTitle: Client Usage
weight: 40
---

After [server installation](/docs/installation/) and [server configuration](/docs/configuration/), you can use the [Web UI](/docs/web-ui/) or run Nebula on devices:

- **[ncclient](/docs/usage/ncclient/)** – Preferred method: device client for installation and usage (Docker, CLI, Windows Tray). Enroll once, then ncclient polls for config and certs and can run Nebula.
- **[nebula](/docs/usage/nebula/)** – Manual alternative: create networks and nodes in the UI, then deploy config and certs yourself and run Nebula manually (e.g. when ncclient is not available, such as on mobile).

All of these assume the backend is running and reachable at the URL you configure.
