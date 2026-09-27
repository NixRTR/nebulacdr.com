---
title: Signing up
linkTitle: Signing up
weight: 10
description: "Create your Nebula Commander Cloud instance and sign in for the first time."
---

Setting up takes a few minutes. Your instance is created automatically as soon as payment goes
through.

## 1. Choose a name and plan

Go to [cloud.nebulacdr.net/signup](https://cloud.nebulacdr.net/signup) and fill in:

- **Email:** this account becomes the administrator of your instance.
- **Organization or project name.**
- **Subdomain:** your instance's address, for example `acme` for `acme.nebulacdr.net`. Use 3–32
  lowercase letters, digits and hyphens. The form tells you straight away if a name is taken or
  reserved.
- **Plan:** *Hosted* at the standard price, or *Name your own price*. For the second, enter your
  monthly amount (at least the minimum shown) or pick one of the suggested amounts.

Accept the Terms of Service, then click **Continue to payment**. Your subdomain is held for you
while you complete checkout.

## 2. Pay

You're taken to Stripe Checkout to enter your card. Nebula Commander Cloud never sees or stores
your card details. If you cancel checkout, you can come back and try again; your subdomain stays
held for a little while.

## 3. Wait a minute or two

After payment you land on a page that updates on its own while your instance is created: its
containers, database, certificate authority, sign-in service and HTTPS certificate. It usually
takes a minute or two. You can close the page; you'll get an email when it's ready.

## 4. Set your password

You'll receive two emails:

1. **"Update your account"**, from the sign-in service. Its link lets you set your password and
   confirm your email address. It's valid for 3 days.
2. **"Your Nebula Commander instance is ready"**, with your instance's address.

Use the first email, then sign in at `https://your-name.nebulacdr.net`. The first time, you're
asked for your first and last name. Your account is the instance **administrator**.

## 5. Next steps

1. [Create a network](/docs/web-ui/networks/) and add a lighthouse: any machine you control with
   a public IP address and UDP port 4242 open.
2. Install the [ncclient](/docs/usage/ncclient/) on your devices and enroll them with codes from
   the [Nodes](/docs/web-ui/nodes/) page.
3. [Invite your team](/docs/web-ui/invitations/). They create their own sign-in on your
   instance's login page.

## "We're not taking new sign-ups right now"

We only accept new instances when the server has room for them. If the sign-up page says it's
full, email [support@nebulacdr.net](mailto:support@nebulacdr.net?subject=Waitlist) and we'll let
you know when a spot opens. In the meantime you can
[self-host Nebula Commander](/docs/installation/) for free, and later
[import](/docs/web-ui/backup/) that instance into Nebula Commander Cloud.
