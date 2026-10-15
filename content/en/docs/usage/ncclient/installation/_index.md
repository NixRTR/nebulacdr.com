---
title: Installation
linkTitle: Installation
weight: 10
---

You can install ncclient in several ways. **For Linux nodes other than lighthouses, the recommended method is the signed package repository at [pkgs.nebulacommander.com](https://pkgs.nebulacommander.com)** ([Linux Desktop App](/docs/usage/ncclient/installation/linux/#package-repository-recommended)): it's the only Linux install that can [update itself automatically](/docs/usage/ncclient/usage/auto-update/). For lighthouses, see Docker below.

| Method                                                              | Best for                                                                 |
|------------------------------------------------------------------------|---------------------------------------------------------------------------|
| [Docker](/docs/usage/ncclient/installation/docker/)                | The first lighthouse in a network, to also run Magic DNS               |
| [Binaries](/docs/usage/ncclient/installation/binaries/)             | Prebuilt CLI binaries from the Web UI or GitHub Releases                |
| [Linux Desktop App](/docs/usage/ncclient/installation/linux/)       | **Recommended for Linux nodes other than lighthouses.** Background service (and optional GTK4 app) from the package repository, with automatic updates |
| [Windows Installer](/docs/usage/ncclient/installation/windows/)     | Windows: CLI, native app, and background service via one MSI           |
| [NixOS](/docs/usage/ncclient/installation/nixos/)                   | NixOS hosts: a declarative `services.ncclient` module                  |
| [Pip (PyPI)](/docs/usage/ncclient/installation/pip/)                | Advanced users only, when none of the above are suitable               |

Choose one and follow the linked page. Then see the matching [Usage](/docs/usage/ncclient/usage/) page to enroll the device, run it, and re-enroll it later.
