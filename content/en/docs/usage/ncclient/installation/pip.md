---
title: Pip (PyPI)
linkTitle: Pip (PyPI)
weight: 50
---

**Fallback method** when Docker, binaries, or NixOS are not suitable (e.g. no Docker, or you need to run from source).

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
