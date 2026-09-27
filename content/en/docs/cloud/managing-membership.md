---
title: Managing your membership
linkTitle: Managing your membership
weight: 20
description: "Billing, changing your price, payment problems, cancelling, and keeping your data."
---

## Signing in to your account

Your account at [cloud.nebulacdr.net](https://cloud.nebulacdr.net/) is where you manage billing.
It's separate from your instance, which you use at `your-name.nebulacdr.net`.

1. Go to [cloud.nebulacdr.net/login](https://cloud.nebulacdr.net/login) and enter the email you
   signed up with.
2. Open the sign-in link we email you. It expires after 15 minutes.

The account page lists your instances with their status and current price.

## Updating your card and downloading invoices

On the account page, click **Manage billing**. This opens Stripe's secure billing page, where you
can:

- update your payment method,
- see and download invoices,
- cancel your subscription.

## Changing your price

Any active instance can use *Name your own price*. On the account page, enter a new monthly
amount under **Name your own price** and click **Update**.

- The amount must be at least the minimum shown. There's no maximum.
- The change applies **from your next invoice**. There's no extra charge or credit for the
  current month.
- Paying more doesn't change the service. It helps fund Nebula Commander's development.

## If a payment fails

- Stripe retries the payment automatically over the following days, and we email you.
- To fix it, update your card under **Manage billing**.
- If payment still hasn't gone through after the retries, the instance is **suspended**. Its
  address shows a "suspended" page and your devices can't fetch new config, but **your data is
  kept**.
- As soon as the payment succeeds, the instance starts again by itself.

## Cancelling

Cancel from **Manage billing**.

1. **Until the end of the period you've paid for**, your instance keeps working as normal.
2. **Then it's suspended.** Its data is kept for **30 days**.
3. **Within those 30 days** you can click **Resubscribe** on the account page to get it back
   exactly as it was.
4. **After 30 days** the instance and its data are permanently deleted. Remaining encrypted
   backups expire as described in the [privacy policy](https://cloud.nebulacdr.net/privacy).

## Keeping your data

Your data is yours, and you can take it with you at any time:

- An instance administrator can download an encrypted export of everything from
  [Backup & export](/docs/web-ui/backup/) in the instance. The export is encrypted with a
  passphrase only you know, so we can't read it.
- **Export before you cancel.** A suspended instance isn't running, so Backup & export isn't
  available then. If your instance is already suspended, email
  [support@nebulacdr.net](mailto:support@nebulacdr.net) from your account's address.
- To move to your own server, [install Nebula Commander](/docs/installation/) and
  [import the export](/docs/web-ui/backup/#importing-into-a-new-instance). With the default
  export options, enrolled devices keep working once you point them at the new server.

## Help

Ask in the
[Nebula Commander Cloud support room on Matrix](https://matrix.to/#/#nebula-commander-cloud-support:matrix.org)
or email [support@nebulacdr.net](mailto:support@nebulacdr.net).
