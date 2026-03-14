---
title: ncclient
linkTitle: ncclient
weight: 20
---

ncclient is the **preferred** way to run Nebula Commander on devices: enroll once with a code from the UI, then ncclient polls for config and certificates and can run or restart [Nebula](https://github.com/slackhq/nebula) when config changes. When you run the first lighthouse using the [Docker client](/docs/usage/ncclient/installation/#docker), you can use [Magic DNS](/docs/web-ui/dns/) (split-horizon DNS) for the network.

- **[Installation](/docs/usage/ncclient/installation/)** – Docker (preferred for first lighthouse), binaries (Web UI or GitHub Releases), Windows Tray (MSI), or Pip (PyPI) as fallback.
- **[Usage](/docs/usage/ncclient/usage/)** – Command line (enrollment, daemon, service) and Windows Tray.

Both the CLI and the Windows tray assume the Nebula Commander backend is running and reachable at the URL you configure.
