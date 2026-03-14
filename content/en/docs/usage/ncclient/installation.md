---
title: ncclient Installation
linkTitle: Installation
weight: 10
---

You can install ncclient in several ways: **Docker** (preferred for the first lighthouse to use [Magic DNS](/docs/web-ui/dns/)), **binaries** (Web UI or GitHub Releases), **Windows MSI** (CLI and tray app), or **Pip (PyPI)** as a fallback when Docker or binaries are not suitable.

## Docker

The Docker client is the **preferred** method for the **first lighthouse** in a network so the container can run dnsmasq and you can use [Magic DNS](/docs/web-ui/dns/) (split-horizon DNS) for the network. Other devices (CLI, tray, or additional Docker clients) can then resolve Nebula hostnames via the lighthouse.

**Image:** `ghcr.io/nixrtr/nebula-commander-ncclient:latest`, or build from the repo `client/docker` (Dockerfile in that directory).

**Required environment:**

- `NEBULA_COMMANDER_SERVER` – Base URL of your Nebula Commander backend (e.g. `https://nc.example.com`), no trailing slash.

**Optional environment:**

- `ENROLL_CODE` – One-time enrollment code from the Nebula Commander UI (Nodes → Enroll for the node). Only used when the token file does not exist; after enrollment the token is stored and this is ignored.
- `SERVE_DNS` – Set to `"true"` to run dnsmasq on this node when it is a lighthouse, so the network can use Magic DNS. Omit or set to `false` if this node is not a lighthouse or you do not need DNS.
- `NEBULA_DNS_POLL_INTERVAL` – Seconds between dnsmasq config polls when this node is a lighthouse (default: 60).
- `NEBULA_OUTPUT_DIR` – Directory where ncclient writes Nebula config and certs inside the container (default: `/data/nebula`).
- `NEBULA_DEVICE_TOKEN_FILE` – Path to the device token file (default: `/data/nebula-commander/token`).

Use a **persistent volume** for `/data` so the token and Nebula config/certs survive restarts. The compose file uses `network_mode: host` so Nebula and dnsmasq can bind to the host.

**Example (docker-compose):**

```yaml
services:
  ncclient:
    image: ghcr.io/nixrtr/nebula-commander-ncclient:latest
    network_mode: host
    restart: unless-stopped
    environment:
      NEBULA_COMMANDER_SERVER: "https://nc.example.com"
      ENROLL_CODE: "XXXXXXXX"   # one-time, from UI
      SERVE_DNS: "true"         # for first lighthouse + Magic DNS
    volumes:
      - ncclient-data:/data

volumes:
  ncclient-data:
    driver: local
```

**Steps:**

1. In Nebula Commander, create a network and add a node for this device. Mark it as a **lighthouse** if this will be the first lighthouse and you want Magic DNS.
2. Create or sign a certificate for the node, then click **Enroll** and copy the one-time code.
3. Set `NEBULA_COMMANDER_SERVER` and `ENROLL_CODE` (and `SERVE_DNS: "true"` for the first lighthouse), then start the container.
4. After enrollment, the container fetches config and certs and runs Nebula (and dnsmasq if `SERVE_DNS` is set and the node is a lighthouse).

## Binaries

If Docker is not an option, use binaries from the Web UI or GitHub Releases.

### From Web UI

When your Nebula Commander instance is deployed with client binaries included (for example the frontend image built with `DOWNLOAD_BINARIES=1`), the Web UI can serve them.

1. Open your Nebula Commander URL in a browser and log in.
2. Go to the downloads or client section (or open `https://YOUR_SERVER/downloads/` if your instance serves that path).
3. Download the binary for your platform:
   - **Linux x86_64:** `ncclient-linux-amd64`
   - **Linux ARM64:** `ncclient-linux-arm64`
   - **Windows x86_64:** `ncclient-windows-amd64.exe`
   - **macOS Intel:** `ncclient-macos-amd64`
   - **macOS Apple Silicon:** `ncclient-macos-arm64`
4. Place the file in a directory on your PATH (or add that directory to PATH). On Linux and macOS, make it executable: `chmod +x ncclient-linux-amd64` (or the file you downloaded).

If your instance does not serve binaries, use [From releases](#from-releases) or [Pip (PyPI)](#pip-pypi) instead.

### From releases

Pre-built binaries are attached to [GitHub Releases](https://github.com/NixRTR/nebula-commander/releases) for each version.

1. Open the [releases page](https://github.com/NixRTR/nebula-commander/releases) and choose a version (e.g. the latest).
2. Download the file for your platform (same names as in [From Web UI](#from-web-ui)).
3. Optionally verify with `SHA256SUMS.txt` in the same release.
4. Place the binary in a directory on your PATH (or add that directory to PATH). On Linux and macOS, make it executable: `chmod +x ncclient-linux-amd64` (or the file you downloaded).

## Windows Installer

On Windows you can install both the ncclient CLI and the optional tray app using the MSI installer.

**What the installer includes:**

- **ncclient** – CLI for enrollment and daemon (poll for config/certs, run or restart Nebula).
- **ncclient-tray** – System tray app: enroll, settings, start/stop polling, optional bundled Nebula, start at login.

Both are installed to `%ProgramFiles%\Nebula Commander\`. The installer can add that directory to PATH and create Start Menu shortcuts.

**Getting the installer:**

- Download `NebulaCommander-windows-amd64.msi` from the [GitHub Releases](https://github.com/NixRTR/nebula-commander/releases) page for the version you want.
- Use `SHA256SUMS.txt` in the same release to verify the file.

**After install:**

1. Enroll: in Nebula Commander go to **Nodes**, open the node, click **Enroll**, and copy the code. Then run:
   ```powershell
   ncclient enroll --server https://YOUR_SERVER_URL --code XXXXXXXX
   ```
2. Run the client from the command line or use the tray app from the Start Menu (see [Windows Tray](/docs/usage/ncclient/usage/#windows-tray) in Usage).

For building the MSI yourself, see [Development: Manual builds](/docs/development/manual-builds/#windows-msi).

## Pip (PyPI)

**Fallback method** when Docker or binaries are not suitable (e.g. no Docker, or you need to run from source).

From PyPI:

```bash
pip install nebula-commander
```

Requires Python 3.10+. This installs the `ncclient` command.

From source (repo clone):

```bash
cd nebula-commander
pip install -r client/requirements.txt
```

Then run as `python -m client --server URL enroll --code XXX`, or install the client in development mode to get the `ncclient` command:

```bash
cd client
pip install -e .
```
