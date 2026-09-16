---
title: Client Download
linkTitle: Client Download
weight: 40
---

The **Client Download** page (also reachable from the sidebar or at `/client-download`) lets users download **ncclient** binaries served directly from your Nebula Commander instance. No internet access to GitHub is required after deployment.

![Client Download page](/screenshots/client-download.png)

## Purpose

Install and run ncclient on a device to enroll with Nebula Commander and pull config and certificates. The enrollment code is obtained from the [Nodes](/docs/web-ui/nodes/) page (Enroll button for the node). This page provides the CLI and, on Windows, the tray app.

## CLI binaries (command-line ncclient)

Pre-built executables for:

| Platform | File name |
|----------|-----------|
| Linux x86_64 | `ncclient-linux-amd64` |
| Linux ARM64 | `ncclient-linux-arm64` |
| Windows x86_64 | `ncclient-windows-amd64.exe` |
| macOS Intel | `ncclient-macos-amd64` |
| macOS Apple Silicon | `ncclient-macos-arm64` |

Downloads are served from `/downloads/` (e.g. `/downloads/ncclient-linux-amd64`). These binaries are only available if the frontend was built with client binaries included (e.g. Docker image built with `DOWNLOAD_BINARIES=1`). If the page shows no downloads or 404, use [ncclient installation from releases](/docs/usage/ncclient/installation/binaries/#from-releases) or pip instead.

After download on Linux or macOS, make the file executable and place it on your PATH:

```bash
chmod +x ncclient-linux-amd64   # or the file you downloaded
# Move to /usr/local/bin or add the directory to PATH
```

On Windows, add the directory containing `ncclient-windows-amd64.exe` to your PATH or run it by full path.

## Windows Tray App

For Windows, use the **MSI installer** (e.g. `/downloads/NebulaCommander-windows-amd64.msi`) - it installs the CLI, the unelevated tray control UI, and a background Windows Service together, and adds them to PATH. The service does the actual work (polling, running Nebula, split-horizon DNS) as `LocalSystem`, so nothing here needs an admin prompt.

The page may also offer the tray and service executables individually:

- **ncclient-tray-windows-amd64.exe** – The tray control UI alone, without the service. Mainly useful for development (see [Windows Tray usage](/docs/usage/ncclient/usage/#windows-tray)) - since the service is only ever registered by the MSI, running this standalone has nothing to control and shows as unreachable. For normal use, install via the MSI instead.
- **ncclient-service-windows-amd64.exe** – The service binary alone. Also mainly for development; normal installs get this through the MSI, which registers it as `NebulaCommanderService`.

## Getting the enrollment code

1. In the Web UI, go to [Nodes](/docs/web-ui/nodes/).
2. Open the node for this device (or create one and create/sign a certificate).
3. Click **Enroll** and copy the one-time code.
4. On the device: if you installed via the Windows MSI, enroll from the **tray app's Enroll dialog** using the server URL and code - the service picks it up automatically. Otherwise (CLI, Docker, other platforms), run: `ncclient enroll --server https://YOUR_SERVER_URL --code XXXXXXXX`, then `ncclient run --server https://YOUR_SERVER_URL` to start polling for config and certs.

See [ncclient usage](/docs/usage/ncclient/usage/) for full steps.
