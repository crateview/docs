---
title: 'Using Crateview'
description: 'Navigate images, switch platforms and inspect the data behind a tag.'
---

## Find repositories and tags

The overview lists repositories from the registry catalog. Explore filters repository names; a `repository:tag` query searches tags within that repository. Nested names such as `team/service` are supported.

Press `/` outside an editable field, or `Ctrl+K`, to focus the navbar search. The sidebar can be collapsed from the toggle beside the search. Language and color-mode controls live at the bottom of the expanded sidebar.

## Inspect an image

Open a repository and select a tag. The image view shows its digest, available platforms, compressed layer sizes and creation metadata.

For a multi-platform image, choose the operating system and architecture in the platform selector. The layer history and selected image config follow that choice. The raw Manifest tab shows the image's root manifest or index; Configuration shows the selected platform's config.

## Pull commands

The command panel provides Docker, Podman and Skopeo commands for images, and ORAS/Podman artifact commands for artifacts. Copy the command and run it in your own terminal.

```sh
docker pull registry.example.com/team/service:latest
podman pull registry.example.com/team/service:latest
oras pull registry.example.com/team/report:latest
```

Replace the example host with your public Crateview origin. CLI authentication is separate from your browser session.

## OCI metadata and referrers

Descriptions, licenses and source links are read from OCI annotations or image config labels when available. Missing metadata stays empty rather than being guessed.

The Referrers tab reads the OCI referrers API. If the registry does not implement that endpoint, Crateview reads the standard digest-derived tag index maintained by OCI clients. It does not build its own index by scanning every manifest. Listed signature artifacts are not automatically verified or marked trusted.

## Interpret timestamps and sizes

Image creation time is provided by the image config or annotations. It does not tell you when a tag was pushed to this registry. Compressed layer sizes do not measure disk consumption, because layers can be shared and deleted data may remain until garbage collection.
