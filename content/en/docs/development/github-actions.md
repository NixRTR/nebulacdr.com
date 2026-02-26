---
title: GitHub Actions
linkTitle: Github Actions
weight: 20
---

Nebula Commander uses two GitHub Actions workflows: one to build ncclient binaries (and create releases), and one to build and push Docker images.

## Build ncclient Binaries

**Workflow file:** `.github/workflows/build-ncclient-binaries.yml`

### Triggers

- **Version tags** – Push a tag matching `v*` (e.g. `v0.1.5`) to build all binaries and create a GitHub Release. This is the only trigger that produces a release; there is no trigger on push to `main`.
- **Pull requests** – Runs when changes touch `client/binaries/**`, `client/windows/**`, `client/ncclient.py`, `installer/windows/**`, or the workflow file itself.
- **Manual** – Use "Run workflow" in the Actions tab (`workflow_dispatch`).

### Platforms built

| Platform | Artifact name |
|----------|----------------|
| Linux x86_64 | `ncclient-linux-amd64` |
| Linux ARM64 | `ncclient-linux-arm64` |
| Windows x86_64 | `ncclient-windows-amd64.exe` |
| macOS Intel | `ncclient-macos-amd64` |
| macOS ARM64 (Apple Silicon) | `ncclient-macos-arm64` |

Windows ARM64 is not built (GitHub has no Windows ARM64 runners). Linux ARM64 is built in Docker with QEMU on the host runner.

### Jobs and outputs

1. **build** – Matrix job: builds the ncclient CLI for each platform. Uploads one artifact per platform.
2. **build-windows-tray** – Builds the Windows tray app (`ncclient-tray.exe`) with optional bundled Nebula. Depends on the CLI build; uploads `ncclient-tray-windows-amd64.exe`.
3. **build-msi** – Runs only on tag pushes. Downloads the Windows CLI and tray artifacts, copies them into `installer/windows/redist/`, builds the MSI with WiX 5, and uploads `NebulaCommander-windows-amd64.msi`.
4. **upload-release** – Runs only on tag pushes. Downloads all artifacts (CLI, tray, MSI), flattens them, generates `SHA256SUMS.txt`, and uploads everything to the GitHub Release for that tag. Release is not draft; files can be overwritten.
5. **checksums** – Generates checksums for the Actions UI; release gets checksums from the upload-release job.
6. **summary** – Prints build status and notes that release binaries were uploaded when the run was tag-triggered.

### Creating a release

1. Bump version (e.g. in `client/pyproject.toml` or as appropriate).
2. Tag and push:
   ```bash
   git tag v0.1.5
   git push origin v0.1.5
   ```
3. The workflow builds all platforms, the tray app, and the MSI, then creates the release and attaches all binaries and SHA256SUMS.

### Manual run

In GitHub: Actions → "Build ncclient Binaries" → "Run workflow". Choose branch and run. Artifacts appear in the run; no release is created unless you ran from a tag.

---

## Build Docker Images

**Workflow file:** `.github/workflows/build-docker-images.yml`

### Triggers

- **After ncclient workflow** – Runs when the "Build ncclient Binaries" workflow completes. It runs only if that workflow **succeeded**. Version is taken from the commit that triggered the binaries workflow: if that commit has a tag `v*`, that tag (without the `v`) is used; otherwise version is `latest`.
- **Manual** – "Run workflow" in the Actions tab. Version is `latest` unless the run is triggered by the binaries workflow.

### Images built and pushed

All images are pushed to GitHub Container Registry (`ghcr.io`):

| Image | Platforms |
|-------|------------|
| `ghcr.io/nixrtr/nebula-commander-backend` | linux/amd64, linux/arm64 |
| `ghcr.io/nixrtr/nebula-commander-frontend` | linux/amd64, linux/arm64 |
| `ghcr.io/nixrtr/nebula-commander-keycloak` | linux/amd64, linux/arm64 |

Each image is tagged with the version (e.g. `1.2.3`) and `latest`. Build uses Docker Buildx, layer caching (GitHub Actions cache), and the Dockerfiles under `docker/`. The frontend image is built with `DOWNLOAD_BINARIES=1` so it can serve ncclient binaries from the release.

### Summary job

A final job prints the version and the full image names with that tag.

---

## Testing workflows locally (act)

You can run the workflows locally with [act](https://github.com/nektos/act). Only **Linux** jobs run in Docker; Windows and macOS jobs do not use real Windows/macOS runners in act. See the repository [.github/README.md](https://github.com/NixRTR/nebula-commander/blob/main/.github/README.md) for setup, usage, and how to simulate a tag push. Release upload and secrets still require GitHub when running with act.
