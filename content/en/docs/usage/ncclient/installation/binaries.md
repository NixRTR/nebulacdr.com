---
title: Binaries
linkTitle: Binaries
weight: 20
---

If Docker is not an option, use binaries from the Web UI or GitHub Releases.

## From Web UI

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

If your instance does not serve binaries, use [From releases](#from-releases) or [Pip (PyPI)](/docs/usage/ncclient/installation/pip/) instead.

## From releases

Pre-built binaries are attached to [GitHub Releases](https://github.com/NixRTR/nebula-commander/releases) for each version.

1. Open the [releases page](https://github.com/NixRTR/nebula-commander/releases) and choose a version (e.g. the latest).
2. Download the file for your platform (same names as in [From Web UI](#from-web-ui)).
3. Optionally verify with `SHA256SUMS.txt` in the same release.
4. Place the binary in a directory on your PATH (or add that directory to PATH). On Linux and macOS, make it executable: `chmod +x ncclient-linux-amd64` (or the file you downloaded).
