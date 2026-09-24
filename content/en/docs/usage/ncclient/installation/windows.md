---
title: Windows Installer
linkTitle: Windows Installer
weight: 30
---

On Windows the MSI installer sets up the ncclient CLI, a native windowed app, and a background Windows Service that does the actual work as `LocalSystem` - no UAC prompts for enrolling, starting/stopping, or applying split-horizon DNS.

![Nebula Commander Windows app](/screenshots/apps/windows-status.png)

**What the installer includes:**

- **ncclient** – CLI for enrollment and daemon (poll for config/certs, run or restart Nebula).
- **Nebula Commander** (WinUI 3) – The windowed desktop app: side tabs for Status, Enrollment, and Settings, minimizes to the tray on close. Talks to the background service.
- **ncclient-service** – The `NebulaCommanderService` Windows Service that polls for config/certs and runs Nebula as `LocalSystem`. Starts automatically at boot; the app talks to it over a local named pipe.

All three are installed to `%ProgramFiles%\Nebula Commander\`. The installer can add that directory to PATH, creates Start Menu shortcuts, and registers/starts the service.

**Getting the installer:**

- Download `NebulaCommander-windows-amd64.msi` from the [GitHub Releases](https://github.com/NixRTR/nebula-commander/releases) page for the version you want.
- Use `SHA256SUMS.txt` in the same release to verify the file.

**After install:**

1. Open **Nebula Commander** from the Start Menu and use the **Enrollment** tab: paste the server URL and the one-time code from Nebula Commander (**Nodes** → open the node → **Enroll**). This is the recommended way to enroll after an MSI install - it writes the token where the service reads it (`%ProgramData%\nebula-commander\`) and immediately notifies the service to fetch config. (The CLI's `ncclient enroll` writes to a separate per-user location the service does not read from, so avoid it for MSI installs unless you've explicitly redirected `NEBULA_COMMANDER_CONFIG_DIR`.)
2. The service starts polling automatically once enrolled - nothing else to run. Use the app's Start/Stop/Restart Service controls on the Status tab, and Settings to change server URL, poll interval, or enable split-horizon DNS. See [Windows App](/docs/usage/ncclient/usage/#windows-app) in Usage for details.

For building the MSI yourself, see [Development: Manual builds](/docs/development/manual-builds/#windows-msi).
