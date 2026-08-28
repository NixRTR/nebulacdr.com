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

When the server has [DNS enabled](/docs/web-ui/dns/) for the network, you can pass `--accept-dns` so ncclient fetches the DNS config (domain and lighthouse IPs) and configures the host to resolve the Nebula domain via the network's DNS. On Linux the client tries, in order: **systemd-resolved**, **dnsmasq**, **NetworkManager**, **systemd-networkd**, then **/etc/resolv.conf** (best-effort). On Windows it uses NRPT. Run as root (Linux) or Administrator (Windows) to apply. To remove the DNS override on exit, stop ncclient normally (e.g. Ctrl+C); the client clears the rules on exit.

**Certificates:** If the cert was **created** via the server (Create certificate in the UI), the bundle includes `host.key`. If it was **signed** (Sign flow), the server does not have the key; put your `host.key` in the same directory as the generated certs (the output dir).

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

Token: `%USERPROFILE%\.config\nebula-commander\token`. Default output dir: `%USERPROFILE%\.nebula`. Use `--nebula` if `nebula.exe` is not on PATH. Do not use `--restart-service`. This plain-CLI token/output location is separate from the shared, service-managed location the tray/service use below - for a GUI and no manual daemon management, use the [Windows Tray](#windows-tray) section below instead.

## Windows Tray

On Windows, the tray app is an **unelevated control UI** for a background **Windows Service** (`NebulaCommanderService`). The service does the actual work - polling for config/certs and running Nebula - as `LocalSystem`, so there is no UAC prompt at any point: not to launch the tray, not to enroll, not to start/stop/restart the daemon, and not to apply split-horizon DNS.

The tray and service are installed together by the [MSI installer](/docs/usage/ncclient/installation/#windows-installer). The tray is not designed to run standalone without it - the service is only ever registered by the MSI (there is no CLI `install`/`remove` subcommand for it), so a standalone tray with no service installed has nothing to control and shows as unreachable.

### Usage

- **Enroll** – Open the tray menu and use Enroll. Enter the server URL and the one-time code from Nebula Commander (Nodes → Enroll for the node). This writes the device token (DPAPI-encrypted, machine-scope) and settings to the shared `%ProgramData%\nebula-commander\` folder the service reads from, then tells the service (over a local named pipe) to poll immediately instead of waiting for the next interval.
- **Settings** – Configure server URL, poll interval, optional path to the Nebula binary, and **Accept split-horizon DNS**. There is no output-directory field - Nebula's config, certs, and logs always live under `%ProgramData%\nebula-commander\`. When the app is built with bundled Nebula, the default Nebula path points to the bundled `nebula.exe`.
- **Start / Stop / Restart Service** – Controls the real Windows Service via the Service Control Manager (not an in-process loop). The installer grants Authenticated Users the rights to do this, so it works with no admin prompt.
- **Run On Startup** – Optional: registers the tray itself (the UI) in the Windows Registry (`HKCU\...\Run`) so the icon appears when you sign in. This only affects the tray UI - the service already starts automatically at boot (`Start="auto"`, `LocalSystem`) regardless of whether anyone is logged in.

Settings are stored in `%ProgramData%\nebula-commander\settings.json` - shared between the tray and the service, not the per-user `%APPDATA%` location older versions used.

### Run from source

From the nebula-commander repo root:

```bash
pip install -r client/windows/requirements.txt
pip install -e client/
python -m client.windows.tray
```

Or with `pythonw` to avoid a console window:

```bash
pythonw -m client.windows.tray
```

Running this way still expects a real installed-and-running `NebulaCommanderService` to control - it won't do anything useful on a machine without one.

### Build (PyInstaller)

To build the standalone tray and service executables (and optionally bundle the Nebula Windows binary):

```bash
cd client/windows
pip install -r requirements.txt pyinstaller
python build.py
```

By default `build.py` builds **both** `ncclient-tray.exe` and `ncclient-service.exe` (`--target both`); pass `--target tray` or `--target service` to build just one. Output is in `client/windows/dist/`. See [client/windows/README.md](https://github.com/NixRTR/nebula-commander/blob/main/client/windows/README.md) and `build.py` for details.

The [Windows MSI installer](/docs/usage/ncclient/installation/#windows-installer) installs and registers all three: `ncclient.exe`, `ncclient-tray.exe`, and `ncclient-service.exe`.
