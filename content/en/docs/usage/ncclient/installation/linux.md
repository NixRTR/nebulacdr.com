---
title: Linux Desktop App
linkTitle: Linux Desktop App
weight: 25
---

On Linux, three packages give you a GTK4/libadwaita desktop app plus a background systemd service that does the actual work - no group membership or relogin needed, and no admin/root prompt for enrolling, starting/stopping, or accepting routes.

| Light | Dark |
|-------|------|
| ![Linux desktop app](/screenshots/apps/linux-status.png) | ![Linux desktop app in dark mode](/screenshots/dark/apps/linux-status.png) |

**What's included:**

- **nebula-commander-client** – The `ncclient` CLI (frozen binary, no Python runtime needed).
- **nebula-commander-service** – Installs `ncclient.service` (systemd), which polls for config/certs and runs Nebula as root. Exposes a system D-Bus API (`org.beardedtek.NebulaCommander1`) that the desktop app talks to, authorized per-call via polkit for any active local session.
- **nebula-commander-desktop** – The GTK4 app: enroll, view connection/service status, and accept or reject offered subnet routes and exit nodes.

Because authorization goes through polkit (`allow_active=yes`) instead of Unix group membership, the desktop app works immediately after install and login - there is no `usermod`/relogin step like older group-based designs.

## Package repository (recommended)

Add the signed Nebula Commander repository once, and new releases arrive with your normal system updates (`apt upgrade`, `dnf upgrade`, `zypper update`). It carries the current and the previous few releases, for amd64 and arm64. Installing `nebula-commander-desktop` and `nebula-commander-service` pulls in `nebula-commander-client`.

**Debian / Ubuntu:**

```bash
sudo install -d -m 0755 /etc/apt/keyrings
curl -fsSL https://pkgs.nebulacommander.com/gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nebula-commander.gpg
sudo curl -fsSL -o /etc/apt/sources.list.d/nebula-commander.sources https://pkgs.nebulacommander.com/deb/nebula-commander.sources
sudo apt update
sudo apt install nebula-commander-desktop nebula-commander-service
```

**Fedora / RHEL:**

```bash
sudo curl -fsSL -o /etc/yum.repos.d/nebula-commander.repo https://pkgs.nebulacommander.com/rpm/nebula-commander.repo
sudo dnf install nebula-commander-desktop nebula-commander-service
```

**openSUSE:**

```bash
sudo zypper addrepo https://pkgs.nebulacommander.com/rpm/nebula-commander.repo
sudo zypper install nebula-commander-desktop nebula-commander-service
```

On a server without a desktop, install just `nebula-commander-service`. The repository metadata and the RPMs are signed with the key at [pkgs.nebulacommander.com/gpg.key](https://pkgs.nebulacommander.com/gpg.key).

The packages below can also be installed directly, without adding the repository.

## .deb (Debian, Ubuntu, and derivatives)

Download all three packages from the [Client Download page](/docs/web-ui/client-download/) or [GitHub Releases](https://github.com/NixRTR/nebula-commander/releases), then install together so `apt` resolves the dependency order:

```bash
sudo apt install ./nebula-commander-client.deb ./nebula-commander-service.deb ./nebula-commander-desktop.deb
```

## .rpm (Fedora, RHEL, and derivatives)

```bash
sudo dnf install ./nebula-commander-client.rpm ./nebula-commander-service.rpm ./nebula-commander-desktop.rpm
```

On openSUSE, use `zypper` instead:

```bash
sudo zypper install ./nebula-commander-*.rpm
```

## Flatpak

The desktop app is also available as a Flatpak bundle. It is not yet published on Flathub, so install the bundle directly:

```bash
flatpak install --user ./org.beardedtek.NebulaCommander.flatpak
```

The Flatpak is the GUI only - it still needs `nebula-commander-service` (or an equivalent backend, such as the [NixOS module](/docs/usage/ncclient/installation/nixos/)) installed and running separately, since a sandboxed Flatpak cannot run Nebula or create a TUN device itself.

## After install

1. Open **Nebula Commander** from your application launcher.
2. Go to the **Enrollment** tab and enter the server URL and the one-time code from Nebula Commander (**Nodes** → open the node → **Enroll**).
3. The **Status** tab shows connection/service state and lets you start/stop/restart the service, view the generated `config.yaml`, and accept or reject offered subnet routes and exit nodes.

See [ncclient usage](/docs/usage/ncclient/usage/) for the equivalent CLI-only steps, or [Development: Manual builds](/docs/development/manual-builds/) to build these packages yourself.
