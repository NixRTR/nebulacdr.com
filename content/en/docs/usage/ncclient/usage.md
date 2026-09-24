---
title: ncclient Usage
linkTitle: Usage
weight: 20
---

After [installing ncclient](/docs/usage/ncclient/installation/), enroll the device once, then run the daemon (or install the service on Linux).

## Command Line

### Enrollment

Enrollment is one-time per device. It stores a device token that ncclient uses to fetch config and certificates.

1. In Nebula Commander, open **Nodes**, select the node for this device, and click **Enroll**.
2. Copy the enrollment code.
3. On the device, run:

```bash
ncclient enroll --server https://YOUR_NEBULA_COMMANDER_URL --code XXXXXXXX
```

The device token is saved to `~/.config/nebula-commander/token` (or `/etc/nebula-commander/token` when run as root). On Windows, the token is stored under `%USERPROFILE%\.config\nebula-commander\token`.

### Run (daemon)

After enrollment, run ncclient so it periodically pulls config and certificates and optionally runs or restarts Nebula:

```bash
ncclient run --server https://YOUR_NEBULA_COMMANDER_URL
```

Defaults: poll every 60 seconds, write files to `/etc/nebula` (or `~/.nebula` on Windows), and start or restart Nebula from PATH when config changes.

#### Options

| Option | Description |
|--------|-------------|
| `--output-dir DIR` | Where to write `config.yaml`, `ca.crt`, `host.crt` (default: `/etc/nebula` on Linux/macOS, `~/.nebula` on Windows) |
| `--interval N` | Poll interval in seconds (default: 60) |
| `--token-file PATH` | Path to device token file |
| `--nebula PATH` | Path to the `nebula` binary if it is not on PATH |
| `--restart-service NAME` | Instead of running nebula directly, restart this systemd service (e.g. `nebula`). Use only one of `--nebula` or `--restart-service`. |
| `--accept-dns` | Enable split-horizon DNS: fetch DNS config from the server and apply it so the Nebula domain is resolved via the network's DNS (lighthouses). On Linux use systemd-resolved, dnsmasq, or similar (run as root); on Windows uses NRPT (run as Administrator). See [Split-horizon DNS](#split-horizon-dns) below. |

Example with nebula in a non-standard location:

```bash
ncclient run --server https://nc.example.com --nebula /usr/local/bin/nebula
```

Example using systemd to run Nebula (ncclient only restarts the service):

```bash
ncclient run --server https://nc.example.com --restart-service nebula
```

**Linux:** Creating the Nebula TUN device requires root. Run ncclient as root, e.g. `sudo ncclient run --server https://...`.

### Split-horizon DNS

When the server has [DNS enabled](/docs/web-ui/dns/) for the network, you can pass `--accept-dns` so ncclient fetches the DNS config (domain and lighthouse IPs) and configures the host to resolve the Nebula domain via the network's DNS. On Linux the client detects how the host manages DNS and configures it to match:

- **systemd-resolved** (on its own, or behind NetworkManager): per-link DNS on the Nebula interface, so only the Nebula domain goes to its servers.
- **NetworkManager without systemd-resolved** (e.g. Debian desktops): NetworkManager's dnsmasq plugin with a rule for the Nebula domain. If NetworkManager is writing `/etc/resolv.conf` itself, the client switches it to the plugin; that needs `dnsmasq` (Debian: `dnsmasq-base`) installed.
- **A standalone dnsmasq service**: a rule in `/etc/dnsmasq.d/`.
- **Plain `/etc/resolv.conf`**: last resort, not true split-horizon.

It also keeps NetworkManager from taking over the Nebula interface, re-checks the DNS on every poll (re-applying it if something undid it), and removes everything when it stops or DNS is turned off. If none of these can work, the client logs why and what to install. On Windows it uses NRPT. Run as root (Linux) or Administrator (Windows) to apply. To remove the DNS override on exit, stop ncclient normally (e.g. Ctrl+C); the client clears the rules on exit.

**Certificates:** If the cert was **created** via the server (Create certificate in the UI), the bundle includes `host.key`. If it was **signed** (Sign flow), the server does not have the key; put your `host.key` in the same directory as the generated certs (the output dir).

### Subnet routes and exit nodes

A [subnet router or exit node](/docs/usage/unsafe-routes/) that an admin assigns to this node is only *offered* to it. It is not used until you accept it on the device. Use `ncclient routes` with the same `--output-dir` that `run` uses:

```bash
ncclient routes list                              # offered routes, and which are accepted
ncclient routes accept 192.168.1.0/24             # add --via <IP> if several gateways offer it
ncclient routes reject 192.168.1.0/24
ncclient routes accept-exit-node --via 10.100.0.1
ncclient routes reject-exit-node
```

You can accept several subnet routes as long as they don't overlap, and at most one exit node. The running daemon picks up changes on its next poll; no restart is needed. The [Linux](#linux-app) and [Windows](#windows-app) apps offer the same controls on their Status pages.

### Install service

#### Linux (quick install)

On Linux you can install a systemd service with one command:

```bash
sudo ncclient install
```

This checks for an existing token at `/etc/nebula-commander/token`. If missing, it prints the exact `ncclient enroll ...` command to run first. It then prompts for server URL and options (output dir, interval, nebula path, restart-service), writes `/etc/default/ncclient` and `/etc/systemd/system/ncclient.service`, and enables (and optionally starts) the service.

- Use `--no-start` to enable without starting.
- Use `--non-interactive` with `NEBULA_COMMANDER_SERVER` (and optional env vars) set for scripting.

#### Other platforms

Run `ncclient run` under your init system (launchd on macOS, Task Scheduler or NSSM on Windows). Example configs are in the repo under `examples/`; see [examples/README-startup.md](https://github.com/NixRTR/nebula-commander/blob/main/client/examples/README-startup.md) for step-by-step setup on macOS and Windows.

### Troubleshooting

- **No TUN device / cannot ping Nebula IP** – On Linux, run with `sudo`. For Sign flow, ensure `host.key` is in the output directory. Check Nebula's error output (e.g. "failed to get tun device", "no such file").
- **Nebula starts then exits** – Often missing `host.key` (Sign flow), wrong config path, or on Linux needing root. Check the Nebula lines ncclient prints.

### macOS notes

Token: `~/.config/nebula-commander/token` (or `/etc/nebula-commander/token` as root). Default output dir: `/etc/nebula`; for non-root use `--output-dir ~/.nebula`. Nebula: use PATH or `--nebula /opt/homebrew/bin/nebula` (Apple Silicon) or `/usr/local/bin/nebula` (Intel). Do not use `--restart-service`; use launchd for background runs.

### Windows notes (CLI)

Token: `%USERPROFILE%\.config\nebula-commander\token`. Default output dir: `%USERPROFILE%\.nebula`. Use `--nebula` if `nebula.exe` is not on PATH. Do not use `--restart-service`. This plain-CLI token/output location is separate from the shared, service-managed location the app/service use below - for a GUI and no manual daemon management, use the [Windows App](#windows-app) section below instead.

## Linux App

| Light | Dark |
|-------|------|
| ![Linux app Status tab](/screenshots/apps/linux-status.png) | ![Linux app Status tab in dark mode](/screenshots/dark/apps/linux-status.png) |

On Linux, **Nebula Commander** (GTK4/libadwaita) is an unelevated desktop app for the background **ncclient** systemd service. The service does the actual work - polling for config/certs and running Nebula - as root, and exposes a system D-Bus API (`org.beardedtek.NebulaCommander1`) that the app talks to, authorized per-call via polkit for any active local session. There is no group membership or relogin step, and no password prompt for enrolling, starting/stopping the service, or accepting routes.

The app and service are installed together via [`.deb`, `.rpm`, or Flatpak](/docs/usage/ncclient/installation/linux/). Like the Windows app, it is not designed to run standalone without the service - if it isn't installed/running, the Status tab reflects that.

### Usage

The app has three tabs in its header bar. It follows the desktop's light/dark preference.

- **Status** – Connection state (e.g. *Connected – Config updated*), the systemd service state with Start/Stop/Restart buttons, a switch for each **subnet route offered to this node**, an **Exit node** picker, and **View Config** (under Advanced) to see the generated `config.yaml`. Routes that overlap one you've already accepted are shown disabled with the reason. You also get a desktop notification when the connection state changes or a new route is offered.
- **Enrollment** – Shows whether the device is already enrolled (and to which server), and lets you enroll with the server URL and the one-time code from Nebula Commander (Nodes → Enroll for the node). Enrolling again replaces the device token.
- **Settings** – Server URL, poll interval, optional path to the Nebula binary (blank uses `PATH`), **Accept split-horizon DNS**, and **Run on startup** (an XDG autostart entry for the app itself). Saving restarts the service automatically so the new values take effect.

| Enrollment | Settings |
|------------|----------|
| ![Linux app Enrollment tab](/screenshots/apps/linux-enroll.png) | ![Linux app Settings tab](/screenshots/apps/linux-settings.png) |

Settings, the device token, and status live under `/var/lib/ncclient/` (root-only; the app never reads these files directly, only through the D-Bus API). The service uses the distribution's own packaged `nebula` binary unless you set a path in Settings.

### Run from source

From the nebula-commander repo root, with PyGObject, GTK 4, libadwaita, and `jeepney` installed (on Debian/Ubuntu: `python3-gi gir1.2-gtk-4.0 gir1.2-adw-1 python3-jeepney`):

```bash
python3 -m client.linux.desktop
```

It still needs a running `ncclient.service` that exposes the D-Bus API (from `nebula-commander-service` or the [NixOS module](/docs/usage/ncclient/installation/nixos/)). Without one, Status shows the service as not installed.

### Build

See [Linux Desktop App installation](/docs/usage/ncclient/installation/linux/) and [Development: Manual builds](/docs/development/manual-builds/) for building the `.deb`/`.rpm`/Flatpak packages yourself.

## Windows App

![Windows app Status page](/screenshots/apps/windows-status.png)

On Windows, **Nebula Commander** (WinUI 3) is a native, **unelevated** windowed app for a background **Windows Service** (`NebulaCommanderService`). The service does the actual work - polling for config/certs and running Nebula - as `LocalSystem`, so there is no UAC prompt at any point: not to launch the app, not to enroll, not to start/stop/restart the daemon, and not to apply split-horizon DNS.

The app and service are installed together by the [MSI installer](/docs/usage/ncclient/installation/windows/). The app is not designed to run standalone without it - the service is only ever registered by the MSI (there is no CLI `install`/`remove` subcommand for it), so a standalone app with no service installed shows Status as unreachable, with nothing to control.

### Usage

The app has three side tabs (Settings is pinned at the bottom of the side bar). It follows the Windows light/dark app theme.

- **Status** – One card each for:
  - **Connection**: the server URL and a **Test Connection** button.
  - **Nebula Commander Service**: state and Start/Stop/Restart.
  - **Nebula Interface**: connected/error, the interface name, whether the Nebula process is running, and when the service last updated.
  - **Split-Horizon DNS**: active or inactive.
  - **Exit Node / Subnet Router**: what this device advertises, plus checkboxes for offered subnet routes and a picker for offered exit nodes.
  - **Configuration**: **View Config** and **Open Containing Folder**.

  Use **Refresh** to update the page right away.
- **Enrollment** – Enter the server URL and the one-time code from Nebula Commander (Nodes → Enroll for the node). If the device is already enrolled, the page says so; enrolling again replaces the token and re-points the device at the new server. Enrolling writes the device token (DPAPI-encrypted, machine-scope) and settings to the shared `%ProgramData%\nebula-commander\` folder the service reads from, then tells the service (over a local named pipe) to poll immediately instead of waiting for the next interval.
- **Settings** – Server URL, poll interval, optional path to the Nebula binary (blank uses `PATH`), and **Split-horizon DNS**. Under **Nebula binary** you can see the installed Nebula version, **Check for updates** against Nebula's GitHub releases, and **Download / update Nebula** into `%ProgramData%\nebula-commander\nebula\`. Under **Startup**, **Run on startup** opens the app to the tray when you sign in. There is no output-directory field - Nebula's config, certs, and logs always live under `%ProgramData%\nebula-commander\`.

| Enrollment | Settings |
|------------|----------|
| ![Windows app Enrollment page](/screenshots/apps/windows-enroll.png) | ![Windows app Settings page](/screenshots/apps/windows-settings.png) |

Closing the window minimizes it to the tray rather than exiting. Use the tray icon's **Open Nebula Commander** item to bring it back, or **Exit** to actually quit the app (the service keeps running either way - it starts automatically at boot, `Start="auto"`, `LocalSystem`, regardless of whether the app is open or anyone is logged in).

Settings are stored in `%ProgramData%\nebula-commander\settings.json` - shared between the app and the service, not a per-user `%APPDATA%` location.

### Run from source

From `client/windows-app/`:

```powershell
dotnet build   # or: dotnet run
```

Running this way still expects a real installed-and-running `NebulaCommanderService` to control - the Status page reflects that if it isn't installed/running yet, rather than failing.

### Build (self-contained publish)

```powershell
cd client/windows-app
dotnet publish -c Release -r win-x64
```

Output: `bin\Release\net10.0-windows*\win-x64\publish\NebulaCommanderApp.exe` - a single self-contained file (bundles the full .NET runtime and Windows App SDK, no prerequisites to install on the target machine). See [client/windows-app/README.md](https://github.com/NixRTR/nebula-commander/blob/main/client/windows-app/README.md) for details.

For the background service, see `client/windows/README.md` and [Development: Manual builds](/docs/development/manual-builds/).

The [Windows MSI installer](/docs/usage/ncclient/installation/windows/) installs and registers all three: `ncclient.exe`, `NebulaCommanderApp.exe`, and `ncclient-service.exe`.
