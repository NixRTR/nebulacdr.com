---
title: Windows App
linkTitle: Windows App
weight: 50
---

![Windows app Status page](/screenshots/apps/windows-status.png)

On Windows, **Nebula Commander** (WinUI 3) is a native windowed app for a background **Windows Service** (`NebulaCommanderService`). The service does the actual work (polling for config and certs, running Nebula, applying split-horizon DNS) as `LocalSystem`.

**Anyone can view; only administrators can change.** Run normally, the app is view-only: status, routes and config are visible to every user. Enrolling, changing settings, accepting routes or exit nodes, installing/updating Nebula, and starting/stopping the service all require an **elevated administrator**. Use the **Relaunch as administrator** bar at the top of the app (or press **Alt+R**; one UAC prompt), or start it with **Run as administrator**. The service enforces this itself on every change, so other users on a shared PC can't re-point or stop your tunnel.

The app and service are installed together by the [MSI installer](/docs/usage/ncclient/installation/windows/). The app isn't designed to run without it: only the MSI registers the service (there's no CLI `install`/`remove` subcommand for it), so a standalone app with no service shows Status as unreachable.

## Where state lives

Everything lives in `%ProgramData%\nebula-commander\`, which only SYSTEM and Administrators can open. The app never touches it directly; it asks the service over a local named pipe.

| What | Where |
|------|-------|
| Device token | `token.bin` (DPAPI-encrypted) |
| Settings | `settings.json`, written by the service |
| Nebula config, certs, and log | `config.yaml` (includes the node's private key), certificates, `nebula.log` |
| Nebula itself | `nebula\`, installed and updated by the service |

The service's own messages go to the Windows **Application** event log.

## Enroll

1. Open **Nebula Commander** from the Start Menu.
2. On the **Enrollment** page, enter the server URL and the one-time code from Nebula Commander (**Nodes** → open the node → **Enroll**).

This needs the app running as administrator. The service makes the enrollment request itself, stores the token, and polls right away instead of waiting for the next interval.

**Don't enroll with `ncclient enroll` from a terminal** on an MSI install. The plain CLI writes to a per-user location the service never reads.

## Using the app

The app has three side tabs (Settings is pinned at the bottom of the side bar) and follows the Windows light/dark app theme.

- **Status**: one card each for:
  - **Connection**: the server URL and a **Test Connection** button.
  - **Nebula Commander Service**: state and Start/Stop/Restart.
  - **Nebula Interface**: connected/error, the interface name, whether the Nebula process is running, and when the service last updated.
  - **Split-Horizon DNS**: active or inactive.
  - **Exit Node / Subnet Router**: what this device advertises, plus checkboxes for offered subnet routes and a picker for offered exit nodes.
  - **Configuration**: **View Config** (the node's private key is shown as `<redacted>`) and **Open Containing Folder** (administrator only).

  Use **Refresh** to update the page right away.
- **Enrollment**: enroll, or see which server the device is enrolled with.
- **Settings**: server URL, poll interval, and **Split-horizon DNS**. Under **Nebula** you can see the installed Nebula version, **Check for updates** against Nebula's GitHub releases, and **Install / update Nebula**: the service downloads the official release, verifies its SHA256 checksum, and installs the whole archive (`nebula.exe`, `nebula-cert.exe`, wintun) itself. The service installs Nebula automatically on first start, and there's no custom Nebula path. Under **Startup**, **Run on startup** opens the app to the tray when you sign in.

| Enrollment | Settings |
|------------|----------|
| ![Windows app Enrollment page](/screenshots/apps/windows-enroll.png) | ![Windows app Settings page](/screenshots/apps/windows-settings.png) |

Closing the window minimizes it to the tray rather than exiting. Use the tray icon's **Open Nebula Commander** item to bring it back, or **Exit** to quit the app. The service keeps running either way: it starts automatically at boot, whether or not the app is open or anyone is signed in.

To control the service without the app, use **Services** (`services.msc`) or an elevated prompt (non-administrators can only query it):

```powershell
sc.exe query NebulaCommanderService
Restart-Service NebulaCommanderService
```

## Subnet routes and exit nodes

Use the checkboxes and the exit-node picker on the **Exit Node / Subnet Router** card of the Status page. The service applies changes right away. See [Subnet routers and exit nodes](/docs/usage/unsafe-routes/) for how routes are offered.

## Split-horizon DNS

Turn on **Split-horizon DNS** in **Settings**. The service applies it with NRPT rules for the network's domain and removes them when turned off or when the service stops. The **Split-Horizon DNS** card on Status shows whether it's active.

## Re-enroll

Open the **Enrollment** page and enroll again with a new code, and a new server URL if you're moving servers. This replaces the token, re-points the device at the server you entered, and makes the service poll immediately. Nothing needs restarting.

## Updates

The service can install new releases by itself during a daily window: **Settings → Updates** (as administrator), or `ncclient auto-update enable` from an elevated prompt. See [Automatic updates](/docs/usage/ncclient/usage/auto-update/).

## Troubleshooting

- **Status shows the service as unreachable**: the MSI's service isn't installed or isn't running. Start it from the Status page or `services.msc`, or reinstall the MSI.
- **"Administrator required"** when changing something: relaunch the app as administrator (the bar at the top of the window).
- **Nebula Interface shows an error**: as administrator, check `%ProgramData%\nebula-commander\nebula.log` (**Open Containing Folder** on Status gets you there).
- **Status says Nebula isn't installed**: the service's first-run download failed (e.g. offline). Use **Settings → Install / update Nebula** as administrator.
- **Enrolled from a terminal and the app still says not enrolled**: the CLI wrote a per-user token. Enroll from the app instead.

## Run from source

From `client/windows-app/`:

```powershell
dotnet build   # or: dotnet run
```

Running this way still expects a real installed-and-running `NebulaCommanderService` to control (the app checks that the control pipe really belongs to that service). If it isn't installed or running, the Status page says so rather than failing.

## Build (self-contained publish)

```powershell
cd client/windows-app
dotnet publish -c Release -r win-x64
```

Output: `bin\Release\net10.0-windows*\win-x64\publish\NebulaCommanderApp.exe`, a single self-contained file (it bundles the .NET runtime and Windows App SDK, so the target machine needs no prerequisites). See [client/windows-app/README.md](https://github.com/NixRTR/nebula-commander/blob/main/client/windows-app/README.md) for details.

For the background service, see `client/windows/README.md` and [Development: Manual builds](/docs/development/manual-builds/). The [Windows MSI installer](/docs/usage/ncclient/installation/windows/) installs and registers all three: `ncclient.exe`, `NebulaCommanderApp.exe`, and `ncclient-service.exe`.
