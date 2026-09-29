---
title: Windows App
linkTitle: Windows App
weight: 30
---

![Windows app Status page](/screenshots/apps/windows-status.png)

On Windows, **Nebula Commander** (WinUI 3) is a native, **unelevated** windowed app for a background **Windows Service** (`NebulaCommanderService`). The service does the actual work (polling for config and certs, running Nebula) as `LocalSystem`, so there's no UAC prompt at any point: not to launch the app, not to enroll, not to start, stop, or restart the service, and not to apply split-horizon DNS.

The app and service are installed together by the [MSI installer](/docs/usage/ncclient/installation/windows/). The app isn't designed to run without it: only the MSI registers the service (there's no CLI `install`/`remove` subcommand for it), so a standalone app with no service shows Status as unreachable.

## Where state lives

Everything lives in the shared `%ProgramData%\nebula-commander\` folder, not in a per-user location:

| What | Where |
|------|-------|
| Device token | `token.bin` (DPAPI-encrypted, machine scope) |
| Settings | `settings.json`, shared by the app and the service |
| Nebula config, certs, and log | `config.yaml`, certificates, `nebula.log` |
| Downloaded Nebula binary | `nebula\` |

The service's own messages go to the Windows **Application** event log.

## Enroll

1. Open **Nebula Commander** from the Start Menu.
2. On the **Enrollment** page, enter the server URL and the one-time code from Nebula Commander (**Nodes** → open the node → **Enroll**).

The app writes the token and settings to `%ProgramData%\nebula-commander\`, then tells the service (over a local named pipe) to poll right away instead of waiting for the next interval.

**Don't enroll with `ncclient enroll` from a terminal** on an MSI install. The plain CLI writes to a per-user location the service never reads.

## Using the app

The app has three side tabs (Settings is pinned at the bottom of the side bar) and follows the Windows light/dark app theme.

- **Status**: one card each for:
  - **Connection**: the server URL and a **Test Connection** button.
  - **Nebula Commander Service**: state and Start/Stop/Restart.
  - **Nebula Interface**: connected/error, the interface name, whether the Nebula process is running, and when the service last updated.
  - **Split-Horizon DNS**: active or inactive.
  - **Exit Node / Subnet Router**: what this device advertises, plus checkboxes for offered subnet routes and a picker for offered exit nodes.
  - **Configuration**: **View Config** and **Open Containing Folder**.

  Use **Refresh** to update the page right away.
- **Enrollment**: enroll, or see which server the device is enrolled with.
- **Settings**: server URL, poll interval, optional path to the Nebula binary (blank uses `PATH`), and **Split-horizon DNS**. Under **Nebula binary** you can see the installed Nebula version, **Check for updates** against Nebula's GitHub releases, and **Download / update Nebula** into `%ProgramData%\nebula-commander\nebula\`. Under **Startup**, **Run on startup** opens the app to the tray when you sign in. There's no output-directory field; Nebula's config, certs, and logs always live under `%ProgramData%\nebula-commander\`.

| Enrollment | Settings |
|------------|----------|
| ![Windows app Enrollment page](/screenshots/apps/windows-enroll.png) | ![Windows app Settings page](/screenshots/apps/windows-settings.png) |

Closing the window minimizes it to the tray rather than exiting. Use the tray icon's **Open Nebula Commander** item to bring it back, or **Exit** to quit the app. The service keeps running either way: it starts automatically at boot, whether or not the app is open or anyone is signed in.

To control the service without the app, use **Services** (`services.msc`) or an elevated prompt:

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

## Troubleshooting

- **Status shows the service as unreachable**: the MSI's service isn't installed or isn't running. Start it from the Status page or `services.msc`, or reinstall the MSI.
- **Nebula Interface shows an error**: check `%ProgramData%\nebula-commander\nebula.log` (**Open Containing Folder** on Status gets you there).
- **Enrolled from a terminal and the app still says not enrolled**: the CLI wrote a per-user token. Enroll from the app instead.

## Run from source

From `client/windows-app/`:

```powershell
dotnet build   # or: dotnet run
```

Running this way still expects a real installed-and-running `NebulaCommanderService` to control. If it isn't installed or running, the Status page says so rather than failing.

## Build (self-contained publish)

```powershell
cd client/windows-app
dotnet publish -c Release -r win-x64
```

Output: `bin\Release\net10.0-windows*\win-x64\publish\NebulaCommanderApp.exe`, a single self-contained file (it bundles the .NET runtime and Windows App SDK, so the target machine needs no prerequisites). See [client/windows-app/README.md](https://github.com/NixRTR/nebula-commander/blob/main/client/windows-app/README.md) for details.

For the background service, see `client/windows/README.md` and [Development: Manual builds](/docs/development/manual-builds/). The [Windows MSI installer](/docs/usage/ncclient/installation/windows/) installs and registers all three: `ncclient.exe`, `NebulaCommanderApp.exe`, and `ncclient-service.exe`.
