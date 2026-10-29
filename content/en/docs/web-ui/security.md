---
title: Security (two-factor login)
linkTitle: Security
weight: 56
description: "Add a second sign-in step with any authenticator app, keep recovery codes, and require it for everyone."
---

From v0.9.0, every account can add **two-factor authentication (2FA)**: after
signing in as usual, Nebula Commander asks for a 6-digit code from an
authenticator app on your phone. Someone who learns your password, or gets into
your identity provider account, still can't get in without that code.

It works with **any authenticator app** that supports standard time-based codes
(TOTP): Google Authenticator, Microsoft Authenticator, Authy, 1Password,
Bitwarden, Aegis, Proton Pass and so on. It also works with any sign-in method
(Keycloak or another OIDC provider, or the standalone dev-token login), because
Nebula Commander checks the code itself.

Two-factor is **optional but strongly recommended**. Accounts without it see a
reminder at the top of the page (dismissing it hides it for 30 days in that
browser). It's **required** to [move a server](/docs/web-ui/move/), and a
server can require it for everyone (see [Requiring it](#requiring-it-for-everyone)).

## Turning it on

1. Open **Security** in the sidebar and click **Set up two-factor**.
2. Scan the QR code with your authenticator app. If you can't scan it, type the
   key shown under it into the app instead.
3. Enter the 6-digit code the app shows to confirm.
4. Save the **10 recovery codes** shown next, in a password manager or on paper,
   then click **I've saved them**. They're shown only once.

From your next sign-in, Nebula Commander asks for a code after your usual
sign-in.

## Signing in

After the usual sign-in, enter the current code from your app. Codes change
every 30 seconds; a code from just before or after is still accepted, so a
slightly slow phone clock doesn't lock you out. Each code works only once.

Lost your phone? Enter one of your **recovery codes** in the same box instead.
Each recovery code works once. When you're running low, the Security page warns
you; click **New recovery codes** to make a fresh set (the old ones stop
working).

After 5 wrong codes in 15 minutes, the account is locked out of code entry for a
while, to stop guessing.

## Turning it off

On the Security page, click **Turn off**, sign in again and enter a current
code. You can't turn it off while the server requires it for everyone, or while
a server move is in progress.

## If someone loses their authenticator

A system admin can reset another user's 2FA on the [Users](/docs/web-ui/users/)
page: open the user and click **Reset 2FA**. This needs the admin to sign in
again and enter **their own** current code. The user then sets up 2FA again at
their next sign-in (or not, if it isn't required). Every reset is written to the
audit log.

## Requiring it for everyone

Set `NEBULA_COMMANDER_REQUIRE_MFA=true` (NixOS: `services.nebula-commander.requireMfa = true;`)
and restart the backend. Then:

- Anyone signing in without 2FA is taken straight to setting it up, and can't use
  Nebula Commander until they've done so.
- Nobody can turn it off.

Nebula Commander Cloud has this turned on.

## Good to know

- 2FA settings aren't part of a [Backup & export](/docs/web-ui/backup/). After
  importing into another server, everyone sets up 2FA again there.
- Two-factor protects sign-ins. It doesn't protect against someone who already
  controls the server itself.
