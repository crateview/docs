---
title: 'Overview'
description: 'What Crateview does, how it connects to your registry, and where its boundaries are.'
---

## A clearer view of your registry

Crateview is a web interface for Docker and OCI container registries. It helps you find repositories and tags, inspect multi-platform images, read manifests and config documents, and understand compressed layers and their history.

It runs alongside your registry. The registry stores images and enforces access control; Crateview supplies the interface and a same-origin proxy. Docker and other OCI clients can use the frontend origin as their registry address.

![Crateview repository overview with Alpine and nested demo repositories.](https://crateview.de/images/docs/v0.2/repositories-light.webp)

## Architecture

```text
Browser / Docker / Podman / ORAS
              |
       Crateview (Nuxt / Nitro)
              |
   Your existing OCI registry
```

The frontend reads the standard Registry HTTP API V2. UI metadata is derived from the catalog, tag lists, manifests and image config blobs. It does not need a separate search index or application database.

## Supported features

- Repository-name search, tag suggestions and nested namespaces.
- Tag tables, platform selection, compressed layer sizes and image history.
- OCI annotations for descriptions, source URLs, documentation and licenses.
- Raw manifest and config JSON with clipboard support.
- OCI referrers, including the standard digest-tag index fallback maintained by OCI clients.
- Anonymous access and native htpasswd Basic Auth.
- Optional manifest deletion, with affected tags shown before confirmation.
- Two interface languages (English and German), System/Light/Dark and configurable branding.


## Know the boundaries

The standard API does not provide push timestamps, download statistics, favorites or vulnerability scanning. Crateview does not fabricate these values. An image creation date comes from image metadata and is not its upload time. Displayed sizes describe compressed image layers, not the registry's disk usage.

External Bearer-token services, OIDC, user administration and API-key management are not integrated. Signature artifacts may be listed as referrers; this is not cryptographic signature verification.

Start with [installation](/docs/v0.2/installation), then review [configuration](/docs/v0.2/configuration) and [authentication](/docs/v0.2/authentication).
