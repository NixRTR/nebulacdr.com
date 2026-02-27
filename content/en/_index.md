---
title: Nebula Commander
description: A self-hosted control plane for Nebula overlay networks
params:
  body_class: td-navbar-links-all-active
---

{{% blocks/cover title="Nebula Commander" height="full td-below-navbar" image_anchor="top" color="dark" %}}

<img src="/logo.svg" alt="Nebula Commander" class="hero-logo mb-4" />

{{% _param description %}}
{.display-6}

<a class="btn btn-lg btn-primary me-3 mb-4" href="/docs/">
  Documentation <i class="fa-solid fa-book ms-2"></i>
</a>
<a class="btn btn-lg btn-info me-3 mb-4" href="https://matrix.to/#/#nebula-commander:matrix.org" target="_blank" rel="noopener">
  Discussion and Support <i class="fa-solid fa-comments ms-2"></i>
</a>
<a class="btn btn-lg btn-secondary mb-4" href="https://github.com/NixRTR/nebula-commander">
  Get the code <i class="fab fa-github ms-2"></i>
</a>

{{% blocks/link-down color="info" %}}

{{% /blocks/cover %}}

{{% blocks/lead color="white" %}}

**Nebula Commander** is a self-hosted control plane for [nebula](https://github.com/slackhq/nebula) overlay networks.

Create networks, manage nodes, allocate IPs, create group firewall rules and issue certificates all from a modern web UI with an optional device client for enrollment and automatic updates.

{{% /blocks/lead %}}

{{% blocks/section color="primary" type="row" %}}

{{% blocks/feature title="Networks & nodes" icon="fa-solid fa-network-wired" %}}

Create and manage Nebula networks, enroll nodes, handle IP allocation, and issue certificates from a single control plane.

{{% /blocks/feature %}}

{{% blocks/feature title="Web UI" icon="fa-solid fa-desktop" %}}

React-based dashboard with OIDC (e.g. Keycloak) or dev token authentication. Full API at `/api` with OpenAPI docs at `/api/docs`.

{{% /blocks/feature %}}

{{% blocks/feature title="Device client (ncclient)" icon="fa-solid fa-terminal" url="/docs/" %}}

`pip install nebula-commander` for enroll and run from scripts or automation. Ideal for fleet and edge deployment.

{{% /blocks/feature %}}

{{% /blocks/section %}}

{{% blocks/section color="white" type="row" %}}

{{% blocks/feature title="NixOS & Docker" icon="fa-solid fa-box" %}}

Run via the NixOS module or Docker. See the [nebula-commander repo](https://github.com/NixRTR/nebula-commander) for installation and configuration.

{{% /blocks/feature %}}

{{% blocks/feature title="Open source" icon="fab fa-github" url="https://github.com/NixRTR/nebula-commander" %}}

Backend and frontend are MIT. Contributions and small, focused PRs are welcome.

{{% /blocks/feature %}}

{{% blocks/feature title="Built for Nebula" icon="fa-solid fa-shield-halved" url="https://github.com/slackhq/nebula" %}}

Nebula is a scalable overlay networking tool. Nebula Commander gives you a central place to operate it.

{{% /blocks/feature %}}

{{% /blocks/section %}}
