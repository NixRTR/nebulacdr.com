---
title: NixOS
linkTitle: NixOS
weight: 20
---

Run Nebula Commander as a NixOS service by adding the module and enabling the service. You can add the module from a local path (clone) or, when available, from a flake.

## Adding the module (path-based, no flake)

Use this when you have the nebula-commander repository on disk (for example under `/etc/nixos` or a path you manage).

### 1. Get the repository

Clone or copy the nebula-commander repo so that the path contains both `nix/` and `backend/`:

```bash
git clone https://github.com/NixRTR/nebula-commander.git /etc/nixos/nebula-commander
# Or use a path of your choice; the module expects ../../backend relative to nix/module.nix
```

### 2. Import the module in your NixOS configuration

In `configuration.nix` (or a NixOS module you include), add the import and enable the service:

```nix
{
  imports = [
    /etc/nixos/nebula-commander/nix/module.nix
  ];

  services.nebula-commander.enable = true;
}
```

If you use a different path, use that path in `imports`, for example `./nebula-commander/nix/module.nix` if the repo is in the same directory as your `configuration.nix`.

### 3. Optional: set options

Override any of the options (see the table below). The default `package` builds the backend from the same repo: it copies `backend/` from the path relative to `nix/module.nix` (`../../backend`), so your clone must have that layout.

```nix
services.nebula-commander = {
  enable = true;
  backendPort = 8081;
  databasePath = "/var/lib/nebula-commander/db.sqlite";
  certStorePath = "/var/lib/nebula-commander/certs";
  jwtSecretFile = null;   # or e.g. /run/secrets/nebula-commander-jwt
  debug = false;
};
```

Then rebuild: `nixos-rebuild switch` (or your usual method).

---

## Adding via a flake

If the nebula-commander repository provides a `flake.nix` that exposes a NixOS module, you can add it as a flake input and use it in your NixOS configuration.

### 1. Add nebula-commander as an input

In your system flake (e.g. `flake.nix` in `/etc/nixos` or your config directory):

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-unstable";
    nebula-commander.url = "github:NixRTR/nebula-commander";
    # If the repo has a flake, you might use a specific ref:
    # nebula-commander.url = "github:NixRTR/nebula-commander/main";
  };

  outputs = { self, nixpkgs, nebula-commander, ... }: {
    nixosConfigurations.yourHost = nixpkgs.lib.nixosSystem {
      system = "x86_64-linux";   # or "aarch64-linux" for ARM, etc.
      modules = [
        ./configuration.nix
        nebula-commander.nixosModules.default   # exact name depends on what the flake exports
      ];
    };
  };
}
```

The exact attribute (e.g. `nebula-commander.nixosModules.default`) depends on what the nebula-commander flake exports. When the flake supports multiple systems, use `x86_64-linux`, `aarch64-linux`, or `aarch64-darwin` as appropriate for your host. If the repository does not yet have a `flake.nix`, use the path-based import above. When a flake is added, it may expose the module under `nixosModules.default` or a named module; check the flake output.

### 2. Enable the service and optional package

In your `configuration.nix` (or in the flake’s module list):

```nix
services.nebula-commander.enable = true;
# If the flake provides a package, point the service at it:
# services.nebula-commander.package = nebula-commander.packages.${pkgs.system}.default;
```

Then rebuild: `nixos-rebuild switch --flake .#yourHost` (or your usual flake command).

---

## Options

All options live under `services.nebula-commander`:

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enable` | bool | — | Enable the Nebula Commander service |
| `package` | package | backend source from repo | Nebula Commander package (backend source). With path-based import, built from `../../backend` relative to the module file. |
| `port` | port | 8080 | Port for the HTTP API when using nginx |
| `backendPort` | port | 8081 | Port for the FastAPI backend (internal) |
| `databasePath` | string | `/var/lib/nebula-commander/db.sqlite` | SQLite database file path |
| `certStorePath` | string | `/var/lib/nebula-commander/certs` | Directory for CA and host certificates |
| `jwtSecretFile` | null or path | null | Path to JWT secret file (e.g. managed by sops-nix). If null, a oneshot service generates `/var/lib/nebula-commander/jwt-secret` on first boot. |
| `debug` | bool | false | Enable debug mode |

The module creates a `nebula-commander` system user and group, tmpfiles for data directories, and (when `jwtSecretFile` is null) a oneshot service that generates a JWT secret. The main service runs uvicorn with the backend and passes environment variables (database URL, cert store path, port, JWT secret file, debug).

For OIDC and other backend settings not exposed as NixOS options, extend the service `environment` in your config or use a config file; the backend reads `NEBULA_COMMANDER_*` from the environment.
