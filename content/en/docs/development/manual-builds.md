---
title: Manual Builds
linkTitle: Manual Builds
weight: 30
---

You can build all ncclient binaries, the Windows service and app, the Windows MSI, the Linux `.deb`/`.rpm`/Flatpak packages, and the Docker images locally without using GitHub Actions.

## ncclient CLI (standalone binary)

The CLI is built with PyInstaller from `client/binaries/`. Python 3.11 is used in CI.

### Same platform (Linux x86_64, Windows x86_64, macOS)

From the repository root:

```bash
pip install -r client/binaries/requirements.txt
pip install -r client/requirements.txt
cd client/binaries
python build.py
```

Output: `client/binaries/dist/ncclient` (or `ncclient.exe` on Windows). Use `python build.py --clean` to remove build artifacts; `python build.py --test` to build and run basic tests.

### Linux ARM64

CI builds Linux ARM64 in a Docker container because the host runner is x86_64. Locally you can do the same:

```bash
docker run --rm --platform linux/arm64 \
  -v "$(pwd):/work" -w /work/client/binaries \
  python:3.11-slim \
  bash -c "
    apt-get update && apt-get install -y binutils &&
    pip install --upgrade pip &&
    pip install -r requirements.txt &&
    pip install -r ../requirements.txt &&
    python build.py
  "
```

The executable will be in `client/binaries/dist/ncclient` (arm64). Run this from the repo root so `$(pwd)` mounts the full tree.

### Windows ARM64

On a Windows ARM64 machine (or with an ARM64 Python), install dependencies and run `python build.py` in `client/binaries`. To force PyInstaller to target ARM64 from an x64 host, set `PYINSTALLER_TARGET_ARCH=arm64` in the environment when running `build.py` (CI does this for the Windows ARM64 matrix; GitHub does not provide Windows ARM64 runners, so this is for local use only).

---

## Windows service

The service (`NebulaCommanderService`) is what actually polls for config/certs and runs Nebula, as `LocalSystem`. It is built with PyInstaller from `client/windows/`.

From the repository root:

```bash
pip install -r client/requirements.txt
pip install -r client/windows/requirements.txt
pip install pyinstaller
cd client/windows
python build.py
```

Output: `client/windows/dist/ncclient-service.exe`. The service does not bundle Nebula: it uses the copy the app downloads into `%ProgramData%\nebula-commander\nebula\`, the path set in Settings, or `nebula.exe` on `PATH`.

Note: `ncclient-service.exe` only does anything useful when registered as a real Windows Service (which the MSI does via WiX's `ServiceInstall`/`ServiceControl` elements) - there is no standalone `install`/`remove` subcommand.

## Windows app (WinUI 3)

The windowed app lives in `client/windows-app/` and needs the .NET 10 SDK (CI uses `10.0.x`). From that directory:

```powershell
dotnet publish -c Release -r win-x64
```

Output: `client/windows-app/bin/Release/net10.0-windows10.0.26100.0/win-x64/publish/NebulaCommanderApp.exe`, a single self-contained file that bundles the .NET runtime and Windows App SDK. See [client/windows-app/README.md](https://github.com/NixRTR/nebula-commander/blob/main/client/windows-app/README.md) for details.

---

## Windows MSI

The MSI installs the ncclient CLI, the Windows app, and the service. You need all three executables and WiX 5.

1. **Get the three executables** – Build as above or download from a release. Copy them into `installer/windows/redist/`:
   - `redist/ncclient.exe` (from `client/binaries/dist/ncclient.exe`)
   - `redist/NebulaCommanderApp.exe` (from the `dotnet publish` output above)
   - `redist/ncclient-service.exe` (from `client/windows/dist/ncclient-service.exe`)

2. **Install WiX 5** – e.g. `dotnet tool install --global wix --version 5.0.2`. Add the Util and UI extensions once:
   ```powershell
   wix extension add -g WixToolset.Util.wixext/5.0.0
   wix extension add -g WixToolset.UI.wixext/5.0.2
   ```

3. **Build the MSI** – From `installer/windows/`:
   ```powershell
   wix build Product.wxs -ext WixToolset.Util.wixext -ext WixToolset.UI.wixext -o NebulaCommander-windows-amd64.msi -d Version=0.5.1 -arch x64
   ```
   Replace `0.5.1` with the version you are building. It must be purely numeric (`major.minor.patch`): strip any pre-release suffix such as `-rc1`. `installer/windows/build-msi.ps1` wraps this and checks that all three files are in `redist/` first.

Output: `NebulaCommander-windows-amd64.msi`. Installing it registers `NebulaCommanderService` (auto-start, `LocalSystem`) and grants Authenticated Users start/stop/query rights on it, so the app's Start/Stop/Restart buttons work without a UAC prompt.

---

## Linux packages (.deb / .rpm)

Both builders produce three packages: `nebula-commander-client` (the frozen CLI, arch-specific), `nebula-commander-service` (systemd unit, D-Bus policy, polkit action/rule; depends on the distro's `nebula` package), and `nebula-commander-desktop` (the GTK4 app as plain Python source; arch: all). From the repository root:

```bash
# .deb - needs dpkg-deb (a Debian/Ubuntu host, container, or WSL)
python3 packaging/deb/build.py --version 0.5.1

# .rpm - needs rpmbuild (Fedora, or the `rpm` package on Debian/Ubuntu)
python3 packaging/rpm/build.py --version 0.5.1
```

The client package uses `client/binaries/dist/ncclient`, building it first if it's missing; pass `--ncclient-binary PATH` to use a binary you already have (CI passes the release's `ncclient-linux-amd64`). `--only desktop service` builds a subset, and `--version` defaults to `git describe`. Output goes to `packaging/deb/dist/` and `packaging/rpm/dist/`.

## Linux Flatpak

The Flatpak contains the desktop app only (it still needs `nebula-commander-service` on the host). It builds against the GNOME 51 runtime:

```bash
flatpak remote-add --if-not-exists --user flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak install --user -y flathub org.gnome.Platform//51 org.gnome.Sdk//51
cd packaging/flatpak
python3 sync-sources.py   # vendor the client sources the manifest builds from
flatpak-builder --force-clean --user --repo=repo --install-deps-from=flathub \
  build-dir org.beardedtek.NebulaCommander.yaml
flatpak build-bundle repo org.beardedtek.NebulaCommander.flatpak org.beardedtek.NebulaCommander
```

Output: `packaging/flatpak/org.beardedtek.NebulaCommander.flatpak`.

---

## Docker

### Backend and frontend (compose)

From the repository root:

```bash
cd docker
docker compose build
```

This builds the backend and frontend images with default build-args. No Keycloak image is built by default; use the Keycloak Dockerfile separately if needed.

### Backend image (docker build)

From the repository root:

```bash
docker build -f docker/backend/Dockerfile -t nebula-commander-backend:local .
```

Optional build-args: `VERSION` (default `latest`), `NEBULA_VERSION` (default `1.8.2`).

### Frontend image (docker build)

From the repository root:

```bash
docker build -f docker/frontend/Dockerfile -t nebula-commander-frontend:local .
```

Build-args:

- **VERSION** – Version tag used when downloading ncclient binaries (default `latest`).
- **DOWNLOAD_BINARIES** – Set to `1` to download ncclient binaries from GitHub releases (by version) into the image; set to `0` (default) for local builds that do not need bundled binaries.

Example with version and binaries:

```bash
docker build -f docker/frontend/Dockerfile \
  --build-arg VERSION=0.1.12 \
  --build-arg DOWNLOAD_BINARIES=1 \
  -t nebula-commander-frontend:0.1.12 .
```

### Keycloak image

From the repository root:

```bash
docker build -f docker/keycloak/Dockerfile -t nebula-commander-keycloak:local .
```

No required build-args. For the nebula login background, ensure `nebula-bg.webp` exists under `docker/keycloak-theme/nebula/login/resources/img/` (or copy from `frontend/public/nebula-bg.webp`) before building.

### Multi-architecture (Buildx)

To build for linux/amd64 and linux/arm64 and push (e.g. to GHCR):

```bash
docker buildx build --platform linux/amd64,linux/arm64 \
  -f docker/backend/Dockerfile \
  -t ghcr.io/nixrtr/nebula-commander-backend:latest \
  --build-arg VERSION=latest \
  --push .
```

Use the same pattern for the frontend (with `VERSION` and `DOWNLOAD_BINARIES` as needed) and keycloak Dockerfiles.
