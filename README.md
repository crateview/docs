# Crateview documentation

Public documentation sources for [Crateview](https://crateview.de).

- Stable: [English](https://crateview.de/docs/v0.4/overview), [Deutsch](https://crateview.de/de/docs/v0.4/overview).
- Archived minors: `v0.1` (0.1.2), `v0.2` (0.2.1), `v0.3` (0.3.0).
- Unreleased preview: `next`; exact source revisions and page lists live in `versions.json`.
- All five versions support English, German, Simplified Chinese, Hindi, Spanish, Arabic, French, Bengali, Brazilian Portuguese, Indonesian, Urdu, Russian and Japanese.

## Editing

Edit `<locale>/docs/<version>/<slug>.md`. Every page has `title` and `description`
frontmatter. English is the editorial reference. Update all translations together,
preserving fenced executable examples and technical identifiers. Additional
translations have not received native-speaker review. Arabic and Urdu render RTL;
code remains LTR. No translation API runs on the website.

Internal links use `/docs/<version>/<slug>` in English and
`/<locale>/docs/<version>/<slug>` otherwise; retain `zh-CN` and `pt-BR` case.
Screenshots use `https://crateview.de/images/docs/<version>/<name>-light.webp`.
They are captured from the matching exact source revision, not borrowed from the
latest interface. Light and dark captures and provenance live in the website
repository. New guide slugs must be added to that version's manifest entry and to
all website locale dictionaries.

## Compatibility review

The initial archive was checked against `v0.1.2`, `v0.2.1`, `v0.3.0`, `v0.4.0`
and preview commit `bef44098e9acfe2aaa39c0ba0bbaa768b7853241`:

- All stable releases expose English/German UI only and use Node 24.22.2.
- 0.1 uses a repository overview and navbar search; 0.2 adds namespace table/grid
  views and the resizable sidebar; 0.3 adds theme-aware code highlighting.
- 0.4 introduces the command palette, bulk digest deletion, optional Distribution
  monitoring, Settings and dependency-license inventory. Monitoring has its own
  guide only where supported; its SQLite database is optional.
- Preview adds file-backed account management, managed token authorization,
  Administration/Users/Account pages, thirteen UI languages, RTL and Node 26.13.0.
  No released image is claimed for this source snapshot.

For each release, check runtime defaults, Compose files, authentication and
permission boundaries, UI routes/shortcuts, deletion semantics and screenshots
against its recorded source revision. Historical docs must not inherit new features
or installation requirements automatically. Keep the released minor archive when
moving the stable marker; update `next` separately. Website tests validate every
version/locale combination, localized links, source revisions and image references.

## Publishing

The private website includes this repository as its `content/` Git submodule.
Commit and push reviewed docs, then update the website's pinned submodule commit,
matching versioned captures and changeset. The website release publishes that exact
revision. Old unversioned URLs redirect temporarily to the selected stable version.
Preview is labeled and excluded from search-engine indexing; on-site search stays
inside the selected version and language.

This repository contains documentation only. Website source, builds and container
image remain private. Registry UI source is public at
[crateview/registry-ui](https://github.com/crateview/registry-ui).
