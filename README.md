# Crateview documentation

Public documentation sources for [Crateview](https://crateview.de).

- English: [crateview.de/docs/overview](https://crateview.de/docs/overview)
- Deutsch: [crateview.de/de/docs/overview](https://crateview.de/de/docs/overview)

Edit `en/docs/*.md` and `de/docs/*.md`. Keep matching slugs and frontmatter (`title`, `description`) in both languages. English internal links use `/docs/...`; German links use `/de/docs/...`.

The private website repository includes this repository as its `content/` Git submodule. Documentation changes appear on the website after a maintainer updates the pinned submodule commit and releases the website. New slugs also require updating the website navigation.

This repository contains documentation only. Website source, builds and its container image are private. The registry UI source is available at [crateview/registry-ui](https://github.com/crateview/registry-ui).
