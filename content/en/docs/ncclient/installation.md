---
title: ncclient Installation
linkTitle: Installation
weight: 10
---

You can install ncclient in several ways: from PyPI with pip, as a standalone binary from the Web UI or GitHub Releases, or on Windows via the MSI installer (CLI and tray app).

## Pip (PyPI)

From PyPI (recommended):

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

## Binaries

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

## Windows Tray

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
2. Run the client from the command line or use the tray app from the Start Menu (see [Windows Tray](/docs/ncclient/usage/#windows-tray) in Usage).

For building the MSI yourself, see [Building and CI: Manual builds](/docs/building-and-ci/manual-builds/#windows-msi).
