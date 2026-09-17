---
title: Users
linkTitle: Users
weight: 55
description: "System-admin user management: view accounts, change system roles, and delete users."
---

The **Users** page (**sidebar → Users**) lists every user account in Nebula
Commander as a grid of square cards. It's visible only to **system admins** —
everyone else never sees this link.

| Light | Dark |
|---|---|
| ![Users page](/screenshots/users.png) | ![Users page in dark mode](/screenshots/dark/users.png) |

## The user grid

Each card shows a user's email, their **system role** badge (`system-admin` in red,
`user` in gray), and how many networks they belong to. Click a card to open its
details.

## User details

The details modal shows the user's email, system role, creation date, and their
per-network permissions — role (owner/member) and the specific capabilities
(manage nodes, invite users, manage firewall) granted on each network they're part
of.

| Light | Dark |
|---|---|
| ![User details modal](/screenshots/users-detail.png) | ![User details modal in dark mode](/screenshots/dark/users-detail.png) |

- **Edit role** — click **Edit** next to the system role to reveal a dropdown
  (`User` or `System Admin`) in place, without leaving the modal. Save applies
  immediately and the modal updates to reflect it.
- **Delete User** — opens a two-step, type-to-confirm delete flow (type the user's
  email to confirm) and requires reauthentication, the same as deleting a network
  or a node. Deleting a user removes all their network permissions; it does not
  delete nodes they created.

System role is separate from per-network permissions: a plain `user` can still be
a network owner with full control over their own networks — `system-admin` is
specifically for instance-wide administration (managing all users, viewing the
audit log across every network, and so on).
