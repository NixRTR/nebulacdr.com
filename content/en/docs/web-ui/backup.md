---
title: Backup & export
linkTitle: Backup & export
weight: 57
description: "Download an encrypted copy of your whole instance, and move it to another server with import."
---

The **Backup & export** page (**sidebar → Backup & export**) lets a system admin
download everything in a Nebula Commander instance as one encrypted file, and
import such a file into a new instance. Use it to:

- keep an offline backup,
- move from hosted Nebula Commander to your own server (or the other way round),
- move a self-hosted instance to new hardware.

Only **system admins** see this page, and both export and import ask you to
**sign in again** before they run.

## What's in an export

Everything needed to run the instance somewhere else:

- networks, their certificate authorities (including the CA private keys) and
  host certificates
- nodes, groups, firewall rules, DNS settings and aliases, IP allocations
- users, their network and node permissions, pending invitations
- the audit log
- optionally, the key that signs device credentials (see
  [Keeping devices working](#keeping-devices-working))

Passwords aren't included. Sign-in is handled by your identity provider
(for example Keycloak), not by Nebula Commander.

## Encryption

When you export, you choose a passphrase (at least 12 characters). The file is
encrypted with it before it leaves the server, in the standard
[age](https://age-encryption.org) format:

- Without the passphrase the file can't be opened, by you, by whoever runs the
  server, or by anyone who gets hold of the file.
- **The passphrase can't be recovered or reset.** After the download, Nebula
  Commander shows it once more (hidden until you click **Show**) so you can save
  it in a password manager.
- The passphrase is never stored or logged. The audit log records that an export
  happened and by whom, not the passphrase.

An export protects your data wherever the file ends up. It can't protect you
from someone who already controls the server running the live instance.

### Opening an export without Nebula Commander

Because it's a standard age file, you can always get at your data:

```sh
age -d -o export.zip nebula-commander-example.com-20260927-1200.ncexport.age
```

Or with the Nebula Commander backend installed:

```sh
python -m backend.scripts.ncexport inspect  export.ncexport.age   # show what's inside
python -m backend.scripts.ncexport decrypt  export.ncexport.age export.zip
```

The zip contains `manifest.json`, one JSON file per table under `data/`, the
certificate store under `certs/` and a `README.txt`. **It contains private
keys**, so treat the decrypted zip like the keys themselves.

## Creating an export

1. Open **Backup & export** and click **Create encrypted export**.
2. Sign in again when asked.
3. Enter a passphrase twice and click **Encrypt and download**.
4. Save the passphrase shown in the next dialog, then click **I've saved it**.

## Importing into a new instance

Import only works on a **fresh** instance, one with no networks or nodes yet.

1. Install Nebula Commander ([Docker](/docs/installation/docker/) or
   [NixOS](/docs/installation/nixos/)) and sign in as a system admin.
2. Open **Backup & export** and click **Import from a file**.
3. Sign in again, choose the export file and enter its passphrase.
4. Check the summary, then point your devices at the new server (below).

Everything is re-encrypted with the new instance's own encryption key. If
anything goes wrong, the import is rolled back and nothing is changed.

### Users and sign-in

- **Same identity provider** (same issuer URL): users keep their accounts and
  just sign in as before.
- **Different identity provider:** each person gets their access back the first
  time they sign in to the new instance with the same email address, **provided
  the provider has verified that address**. In Keycloak, turn on
  **Realm settings → Login → Verify email** before inviting people in. This
  check stops someone who registers with another person's unverified email from
  inheriting their access.
- The admin who runs the import is matched to their old account by email
  straight away.

### Keeping devices working

With **Keep enrolled devices working after a move** ticked (the default), the
export includes the key that signed your devices' credentials. After importing,
each device only needs the new server address:

```sh
# /etc/default/ncclient (Linux service) or the ncclient environment
NEBULA_COMMANDER_SERVER=https://nebula.example.com
```

Restart ncclient. The device carries on with its existing certificate and keeps
receiving config updates, with no re-enrollment. The imported key is stored
encrypted and is only ever accepted for device credentials, never for signing
in. New credentials are always issued with the new instance's own key.

Leave the box unticked for a plain backup you won't import elsewhere. Devices
would then need a new enrollment code after an import.
