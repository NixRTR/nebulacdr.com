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
# Or use a path of your choice; the module expects ../backend relative to nix/module.nix
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

Override any of the options (see the table below). The default `package` builds the backend from the same repo: it copies `backend/` from the path relative to `nix/module.nix` (`../backend`), so your clone must have that layout.

```nix
services.nebula-commander = {
  enable = true;
  backendPort = 8081;
  databasePath = "/var/lib/nebula-commander/db.sqlite";
  certStorePath = "/var/lib/nebula-commander/certs";
  jwtSecretFile = null;   # or e.g. /run/secrets/nebula-commander-jwt
  encryptionKeyFile = null;   # or e.g. /run/secrets/nebula-commander-encryption-key
  debug = false;

  # Production/OIDC options (all optional):
  publicUrl = "https://nebula.example.com";
  corsOrigins = "https://nebula.example.com";
  sessionHttpsOnly = true;
  oidc = {
    issuerUrl = "https://keycloak.example.com/realms/nebula-commander";
    clientId = "nebula-commander";
    clientSecretFile = "/run/secrets/nebula-commander-oidc-secret";
  };
};
```

Then rebuild: `nixos-rebuild switch` (or your usual method).

---

## Adding via a flake

The nebula-commander repository's `flake.nix` exposes `nixosModules.default` for the server (this page) plus `packages.default`/`backend`/`frontend`. It also exposes `nixosModules.client` and `packages.ncclient` for the device client — see [ncclient Installation: NixOS](/docs/usage/ncclient/installation/nixos/) if you also want to run the client declaratively, on this host or another.

### 1. Add nebula-commander as an input

In your system flake (e.g. `flake.nix` in `/etc/nixos` or your config directory):

```nix
{
  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-25.11";
    nebula-commander.url = "github:NixRTR/nebula-commander";
  };

  outputs = { self, nixpkgs, nebula-commander, ... }: {
    nixosConfigurations.yourHost = nixpkgs.lib.nixosSystem {
      system = "x86_64-linux";   # or "aarch64-linux" for ARM, etc.
      modules = [
        ./configuration.nix
        nebula-commander.nixosModules.default
      ];
    };
  };
}
```

If the repository does not yet have a `flake.nix` at the ref you're pinning, use the path-based import above instead.

### 2. Enable the service and optional package

In your `configuration.nix` (or in the flake's module list):

```nix
services.nebula-commander.enable = true;
# If the flake provides a package, point the service at it:
# services.nebula-commander.package = nebula-commander.packages.${pkgs.system}.default;
```

Then rebuild: `nixos-rebuild switch --flake .#yourHost` (or your usual flake command).

---

## Server options

All options live under `services.nebula-commander`:

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `enable` | bool | — | Enable the Nebula Commander service |
| `package` | package | backend source from repo | Nebula Commander package (backend source). With path-based import, built from `../backend` relative to the module file. |
| `port` | port | 8080 | Port for the HTTP API when using nginx |
| `backendPort` | port | 8081 | Port for the FastAPI backend (internal) |
| `databasePath` | string | `/var/lib/nebula-commander/db.sqlite` | SQLite database file path |
| `certStorePath` | string | `/var/lib/nebula-commander/certs` | Directory for CA and host certificates |
| `jwtSecretFile` | null or path | null | Path to JWT secret file (e.g. managed by sops-nix). If null, a oneshot service generates `/var/lib/nebula-commander/jwt-secret` on first boot. |
| `encryptionKeyFile` | null or path | null | Path to the Fernet encryption-at-rest key (e.g. managed by sops-nix). If null, a oneshot service generates `/var/lib/nebula-commander/encryption-key` on first boot. |
| `debug` | bool | false | Enable debug mode |
| `publicUrl` | null or string | null | Public URL of this instance (e.g. `https://nebula.example.com`). Derives the OIDC redirect URI and is used for redirect validation. |
| `standaloneAdminBootstrap` | bool | false | Allow the unauthenticated dev-token admin bootstrap endpoint when no OIDC provider is configured. Must be explicitly opted into. |
| `corsOrigins` | string | `"*"` | CORS origins: `"*"` or a comma-separated list. |
| `sessionHttpsOnly` | bool | false | Set to true in production when served over HTTPS. |
| `allowedRedirectHosts` | string | `""` | Comma-separated allowed hosts for OAuth/OIDC redirects. Empty derives from `oidc.redirectUri`/`publicUrl`. |
| `oidc.issuerUrl` | null or string | null | OIDC issuer URL (e.g. a Keycloak realm URL). Leave null to use dev-token auth instead. |
| `oidc.publicIssuerUrl` | null or string | null | Public-facing OIDC issuer URL for browser redirects (logout, etc.), if different from `issuerUrl`. |
| `oidc.clientId` | null or string | null | OIDC client ID. |
| `oidc.clientSecretFile` | null or path | null | Path to a file containing the OIDC client secret (e.g. managed by sops-nix). |
| `oidc.redirectUri` | null or string | null | OIDC redirect URI. If unset and `publicUrl` is set, derived as `publicUrl + /api/auth/callback`. |
| `oidc.scopes` | string | `"openid profile email"` | OIDC scopes to request. |

The module creates a `nebula-commander` system user and group, tmpfiles for data directories, and oneshot services that generate the JWT secret and encryption key when their `*File` options are left null. The main service runs uvicorn with the backend and passes all of the above through as `NEBULA_COMMANDER_*` environment variables.

For settings not yet exposed as NixOS options, extend the service `environment` in your config; the backend reads any `NEBULA_COMMANDER_*` variable from the environment — see [Environment variables](/docs/configuration/environment/) for the full list.

Looking to run the **device client** (`ncclient`) declaratively on NixOS instead of (or alongside) this server? See [ncclient Installation: NixOS](/docs/usage/ncclient/installation/nixos/).
