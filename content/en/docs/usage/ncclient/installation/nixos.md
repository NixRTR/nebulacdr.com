---
title: NixOS
linkTitle: NixOS
weight: 40
---

A `services.ncclient` module runs ncclient declaratively as a systemd service — an alternative to the Docker image, the Windows service, or a hand-written unit (see [ncclient CLI: Run as a service](/docs/usage/ncclient/usage/cli/#run-as-a-service)). It's exposed by the same `flake.nix` as the [server module](/docs/installation/nixos/#adding-via-a-flake), as `nixosModules.client`.

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
  # One-time enrollment: a file holding the enrollment code, used once (the unit
  # records that the network is enrolled and doesn't send the code again).
  enrollCodeFile = "/run/secrets/ncclient-enroll-code";
  acceptDns = true;
};
```

Then rebuild: `nixos-rebuild switch` (path-based) or `nixos-rebuild switch --flake .#yourHost`. See [NixOS usage](/docs/usage/ncclient/usage/nixos/) for controlling the service, running `ncclient` commands, and re-enrolling.

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enable` | bool | — | Enable the ncclient service |
| `package` | package | built from `nix/client-package.nix` | The ncclient package |
| `nebulaPackage` | package | `pkgs.nebula` | Package providing the `nebula`/`nebula-cert` binaries ncclient orchestrates |
| `server` | null or string | null | Nebula Commander server URL, for a device in one network. With several networks use `networks` instead. |
| `enrollCodeFile` | null or path | null | Path to a file containing a one-time enrollment code for `server`. A oneshot unit runs `ncclient enroll` once, before the main service starts, and records that it did (same as `networks.default`). Leave null to enroll by hand or provision the token out of band. |
| `networks` | attribute set of `{ server; enrollCodeFile; }` | `{ }` | From v0.8.0: networks this device joins, each with its own server and code file, enrolled once each. Removing an entry doesn't leave the network; use `ncclient networks remove`. See [Several networks](/docs/usage/ncclient/usage/nixos/#several-networks). |
| `interval` | int | 60 | Poll interval in seconds |
| `outputDir` | string | `/var/lib/ncclient/nebula` | Directory ncclient writes Nebula's config/certs/binary to |
| `acceptDns` | bool | false | Accept and apply DNS settings pushed by Nebula Commander |
| `adminGroups` | null or list of strings | `[ "wheel" "sudo" ]` | Active local users in these groups may change the device (desktop app enrollment, settings, routes, exit node; start/stop/restart `ncclient.service`) without a password. Everyone else needs an administrator's password. `null` lets any active local user do it (single-user machines only). |
| `stateDir` | string | `/var/lib/ncclient` | Directory holding the device tokens and `settings.json` together on the same persistent path — both must live in the same place or the node's identity is lost on restart even though the token survives |

The service runs as root, matching the Windows Service (`LocalSystem`) and Docker image (root-in-container) precedent above — Nebula needs to create a TUN device. `--nebula`/`--restart-service` are intentionally not exposed; `nebula` is resolved via `PATH` (from `nebulaPackage`), matching the Docker image's approach.

## Desktop app

On a NixOS desktop, you can add the [Linux desktop app](/docs/usage/ncclient/usage/linux/) alongside the service with the `nixosModules.client-desktop` module (or `nix/client-desktop-module.nix` by path):

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

The desktop module installs the app and turns on `security.polkit.enable` (as a default you can override), since every request the app makes is authorized by polkit. `services.ncclient` registers the D-Bus policy and polkit rules the app needs. Any user in an active local session can view status; members of `adminGroups` (default `wheel`/`sudo`) can enroll, change settings, accept routes, and start/stop/restart `ncclient.service` without a password prompt, and everyone else needs an administrator's password.
