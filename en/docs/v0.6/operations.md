---
title: 'Operations & deletion'
description: 'Manage manifests carefully and keep registry storage under your control.'
---

## Enable manifest deletion

Deletion is disabled in the Crateview UI by default. Both the frontend and Distribution must allow it:

```dotenv
NUXT_REGISTRY_DELETE_ENABLED=true
```

```yaml
storage:
  delete:
    enabled: true
```

Distribution can also read `REGISTRY_STORAGE_DELETE_ENABLED=true`. Restart the relevant services after changing their environment or configuration. The registry remains responsible for authorization, including requests made directly through `/v2/**`.

## Understand digest deletion

The standard delete operation targets a **manifest digest**. Every tag pointing to that digest can be removed. Crateview previews the affected tags before confirmation, but a concurrent writer can change references after the preview: the registry API does not provide a transaction that locks aliases.

Treat a deletion as a change to the manifest and all its aliases, not as removal of one label. Verify your backups and intended scope before confirming.


Select one or more tags in the repository table and confirm deletion. The preview refreshes affected tags, shared digests are deleted once, and partial failures are reported. All aliases of a deleted digest are affected. Garbage collection remains a separate registry task.

## Garbage collection and backups

Deleting a manifest does not promise immediate disk reclamation. Garbage collection is a separate registry administration task; Crateview does not run it.

Back up your registry storage and its configuration using a method appropriate to the storage driver. Keep volumes when replacing containers. Follow Distribution's instructions for garbage collection and write coordination:

- [Distribution garbage collection](https://distribution.github.io/distribution/about/garbage-collection/)
- [Distribution configuration](https://distribution.github.io/distribution/about/configuration/)

## Update Crateview

Review the release notes, set `CRATEVIEW_VERSION` to the desired `v`-prefixed release tag in `.env`, and replace the frontend container:

```sh
docker compose pull frontend
docker compose up -d --no-deps frontend
```

For example, the initial release uses `CRATEVIEW_VERSION=v0.6.0`. To roll back, restore the previous tag and repeat these commands. Keep the runtime environment and `registry-data` volume; do not run `docker compose down --volumes`. Source builds remain available through `compose.build.yaml`.

Dependabot is configured in the source repository to propose dependency updates; it does not upgrade your self-hosted deployment automatically. Re-test registry connectivity, sign-in and affected actions after an upgrade.

## Large registries

Repository search reads the complete paginated catalog. Metadata for visible repositories is fetched on demand. Sorting a large tag list by image creation time can require reading many manifests and configs. Crateview has no background metadata index or external search service.
