---
title: ncclient
linkTitle: ncclient
weight: 20
---

**Experimental:** ncclient is experimental and will change often. Using [Nebula manually](/docs/usage/nebula/) is the stable method for now.

ncclient is the Nebula Commander device client. It enrolls once with a code from the UI, then polls for config and certificates and can run or restart [Nebula](https://github.com/slackhq/nebula) when config changes.

- **[Installation](/docs/usage/ncclient/installation/)** – Pip (PyPI), binaries (Web UI or GitHub Releases), or Windows Tray (MSI).
- **[Usage](/docs/usage/ncclient/usage/)** – Command line (enrollment, daemon, service) and Windows Tray.

Both the CLI and the Windows tray assume the Nebula Commander backend is running and reachable at the URL you configure.
