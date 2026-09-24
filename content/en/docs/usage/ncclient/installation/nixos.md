---
title: NixOS
linkTitle: NixOS
weight: 40
---

A `services.ncclient` module runs ncclient declaratively as a systemd service — an alternative to the Docker image, the Windows service, or a hand-written unit (see [Install service](/docs/usage/ncclient/usage/#install-service) in Usage). It's exposed by the same `flake.nix` as the [server module](/docs/installation/nixos/#adding-via-a-flake), as `nixosModules.client`.

```nix
{
  inputs.nebula-commander.url = "github:NixRTR/nebula-commander";

  outputs = { self, nixpkgs, nebula-commander, ... }: {
    nixosConfigurations.yourHost = nixpkgs.lib.nixosSystem {
      system = "x86_64-linux";
      modules = [
        ./configuration.nix
        nebula-commander.nixosModules.client
      ];
    };
  };
}
```

Or import `nix/client-module.nix` directly by path if you're not using a flake, the same way the [server module](/docs/installation/nixos/#adding-the-module-path-based-no-flake) can be.

```nix
services.ncclient = {
  enable = true;
  server = "https://nebula.example.com";
  # One-time enrollment: a file holding the enrollment code, consumed once (the
  # unit only runs `ncclient enroll` when no token exists yet at stateDir/token).
  enrollCodeFile = "/run/secrets/ncclient-enroll-code";
  acceptDns = true;
};
```

Then rebuild: `nixos-rebuild switch` (path-based) or `nixos-rebuild switch --flake .#yourHost`.

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enable` | bool | — | Enable the ncclient service |
| `package` | package | built from `nix/client-package.nix` | The ncclient package |
| `nebulaPackage` | package | `pkgs.nebula` | Package providing the `nebula`/`nebula-cert` binaries ncclient orchestrates |
| `server` | string | — (required) | Nebula Commander server URL |
| `enrollCodeFile` | null or path | null | Path to a file containing a one-time enrollment code. When set and no token exists yet, a oneshot unit runs `ncclient enroll` before the main service starts. Leave null if you provision the token file out of band. |
| `interval` | int | 60 | Poll interval in seconds |
| `outputDir` | string | `/var/lib/ncclient/nebula` | Directory ncclient writes Nebula's config/certs/binary to |
| `acceptDns` | bool | false | Accept and apply DNS settings pushed by Nebula Commander |
| `stateDir` | string | `/var/lib/ncclient` | Directory holding the device token and `settings.json` together on the same persistent path — both must live in the same place or the node's identity is lost on restart even though the token survives |

The service runs as root, matching the Windows Service (`LocalSystem`) and Docker image (root-in-container) precedent above — Nebula needs to create a TUN device. `--nebula`/`--restart-service` are intentionally not exposed; `nebula` is resolved via `PATH` (from `nebulaPackage`), matching the Docker image's approach.

## Desktop app

On a NixOS desktop, you can add the [Linux desktop app](/docs/usage/ncclient/usage/#linux-app) alongside the service with the `nixosModules.client-desktop` module (or `nix/client-desktop-module.nix` by path):

```nix
modules = [
  ./configuration.nix
  nebula-commander.nixosModules.client
  nebula-commander.nixosModules.client-desktop
];
```

```nix
services.ncclient.enable = true;          # the service the app controls
services.ncclient-desktop.enable = true;  # adds "Nebula Commander" to the app launcher
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enable` | bool | — | Install the Nebula Commander desktop app (GTK4/libadwaita) |
| `package` | package | built from `nix/client-desktop-package.nix` | The desktop app package |

The desktop module only installs the app. `services.ncclient` already registers the D-Bus policy and polkit rules the app needs, so any user in an active local session can enroll, change settings, accept routes, and start/stop/restart `ncclient.service` without a password prompt or group membership.
