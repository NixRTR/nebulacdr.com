---
title: Manual Builds
linkTitle: Manual Builds
weight: 30
---

You can build all ncclient binaries, the Windows tray app, the Windows MSI, and the Docker images locally without using GitHub Actions.

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

## Windows tray app

From the repository root:

```bash
pip install -r client/requirements.txt
pip install -r client/windows/requirements.txt
pip install pyinstaller
cd client/windows
python build.py
```

Output: `client/windows/dist/ncclient-tray.exe`. The build can optionally bundle the Nebula Windows binary; see `client/windows/README.md` and `build.py` for details.

---

## Windows MSI

The MSI installs the ncclient CLI and the tray app. You need both executables and WiX 5.

1. **Get the two executables** – Build as above or download from a release. Copy them into `installer/windows/redist/`:
   - `redist/ncclient.exe` (from `client/binaries/dist/ncclient.exe`)
   - `redist/ncclient-tray.exe` (from `client/windows/dist/ncclient-tray.exe`)

2. **Install WiX 5** – e.g. `dotnet tool install --global wix --version 5.0.2`. Add the Util extension once:
   ```powershell
   wix extension add -g WixToolset.Util.wixext/5.0.0
   ```

3. **Build the MSI** – From `installer/windows/`:
   ```powershell
   wix build Product.wxs -ext WixToolset.Util.wixext -o NebulaCommander-windows-amd64.msi -d Version=0.1.12 -arch x64
   ```
   Replace `0.1.12` with the version you are building.

Output: `NebulaCommander-windows-amd64.msi`.

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
