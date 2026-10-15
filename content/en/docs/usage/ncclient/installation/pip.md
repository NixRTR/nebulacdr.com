---
title: Pip (PyPI)
linkTitle: Pip (PyPI)
weight: 50
---

**For advanced users only.** Pip is the most delicate way to install ncclient. It's easy to end up with a setup that `sudo` can't find, that root can't safely run, or that stores the token somewhere the service doesn't look. Use the [Linux packages](/docs/usage/ncclient/installation/linux/), the [Windows installer](/docs/usage/ncclient/installation/windows/), [Docker](/docs/usage/ncclient/installation/docker/), or the [binaries](/docs/usage/ncclient/installation/binaries/) unless you need to run from source or none of those fit.

From PyPI:

```bash
pip install nebula-commander
```

Requires Python 3.10+. This installs the `ncclient` command.

From source (repo clone):

```bash
cd nebula-commander
pip install -r client/requirements.txt
```

Then run as `python -m client --server URL enroll --code XXX`, or install the client in development mode to get the `ncclient` command:

```bash
cd client
pip install -e .
```

## Linux

On Debian, Ubuntu, Fedora, and openSUSE, the [packages](/docs/usage/ncclient/installation/linux/) are easier: they install a service that manages all of this for you. Use pip when packages aren't an option.

On Linux, ncclient runs as root (Nebula needs root to create its TUN interface), so install it where root owns it and `sudo` can find it. Don't point the service at a virtualenv in your home directory. Anything your user can modify would then run as root.

```bash
sudo python3 -m venv /opt/ncclient
sudo /opt/ncclient/bin/pip install nebula-commander
sudo ln -s /opt/ncclient/bin/ncclient /usr/local/bin/ncclient
```

This also works on distributions where a plain `sudo pip install` is blocked ("externally-managed-environment"). To upgrade later, run `sudo /opt/ncclient/bin/pip install --upgrade nebula-commander`, then `sudo systemctl restart ncclient`.

If `sudo ncclient` says `command not found`, your distribution's `sudo` doesn't include `/usr/local/bin` in its `PATH` (RHEL and its rebuilds). Use the [rpm package](/docs/usage/ncclient/installation/linux/) there instead.

Next, follow [Linux setup](/docs/usage/ncclient/usage/cli/#linux-setup): enroll as root, then install the service.

## Other platforms

Next, see [ncclient CLI usage](/docs/usage/ncclient/usage/cli/) to enroll and run it.
