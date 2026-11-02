# Crateview documentation

Public documentation sources for [Crateview](https://crateview.de).

- English: [crateview.de/docs/overview](https://crateview.de/docs/overview)
- Deutsch: [crateview.de/de/docs/overview](https://crateview.de/de/docs/overview)
- Also available: 简体中文 (`zh-CN`), हिन्दी (`hi`), Español (`es`), العربية (`ar`), Français (`fr`), বাংলা (`bn`), Português (Brasil) (`pt-BR`), Bahasa Indonesia (`id`), اردو (`ur`), Русский (`ru`), 日本語 (`ja`).

Edit `<locale>/docs/*.md`. All 13 locales contain the same eight guide slugs and frontmatter (`title`, `description`). English internal links use `/docs/...`; all other languages use `/<locale>/docs/...`, preserving the case in `zh-CN` and `pt-BR`.

English is the editorial reference. German translations are maintained separately; the eleven additional languages initially use machine translation and have not received native-speaker review. When changing a guide, update every translation, preserve fenced code examples and inline technical identifiers, and localize internal link targets. Website tests check guide coverage, links and code examples. Arabic and Urdu use RTL presentation; technical examples remain LTR. No translation service is called by the deployed website.

The private website repository includes this repository as its `content/` Git submodule. Documentation changes appear on the website after a maintainer updates the pinned submodule commit and releases the website. New slugs also require updating the website navigation.

This repository contains documentation only. Website source, builds and its container image are private. The registry UI source is available at [crateview/registry-ui](https://github.com/crateview/registry-ui).
