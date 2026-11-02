---
title: 'Configuration'
description: 'Registry connection, runtime environment variables and deployment behavior.'
---

## Configuration layers

Application defaults are defined in `nuxt.config.ts` and `app/app.config.ts`. Environment variables override runtime defaults when the server starts. Development and build commands read `.env`; the production Node entrypoint does not read it automatically.

Restart the server after changing runtime environment variables. Editing source-level component configuration requires a new production build.

## Registry and session settings

| Variable                       | Default                 | Purpose                             |
| ------------------------------ | ----------------------- | ----------------------------------- |
| `NUXT_REGISTRY_UPSTREAM`       | `http://127.0.0.1:5000` | Server-side registry origin.        |
| `NUXT_REGISTRY_DELETE_ENABLED` | `false`                 | Expose manifest deletion in the UI. |
| `NUXT_SESSION_PASSWORD`        | Random per process      | Secret used to encrypt UI sessions. |
| `HOST`                         | Runtime-dependent       | Node server listening interface.    |
| `PORT`                         | `3000`                  | Node server listening port.         |

The upstream must be an HTTP(S) origin reachable from the frontend server. Keep credentials out of the URL. Requests use the current user's authentication; Crateview does not replace registry access control.

## Public branding variables

All names below use the `NUXT_PUBLIC_BRAND_` prefix:

| Suffix                                      | Purpose                                         |
| ------------------------------------------- | ----------------------------------------------- |
| `NAME`                                      | Application name.                               |
| `DESCRIPTION`                               | Application description.                        |
| `LOGO_LIGHT`, `LOGO_DARK`                   | Logo paths or HTTP(S) URLs.                     |
| `FAVICON`                                   | Browser icon path or URL.                       |
| `TITLE_TEMPLATE`                            | Page title; `%s` is replaced by the page title. |
| `PRODUCT_URL`, `DOCS_URL`, `REPOSITORY_URL` | Optional navigation links.                      |

Public configuration is sent to the browser. Never store passwords or tokens in `NUXT_PUBLIC_*` variables. See [branding and theming](/docs/v0.4/theming) for color and typography options.

## Network layout

The browser communicates with Crateview on one origin. Nitro proxies `/v2/**` for registry clients and reads registry metadata through `/api/registry/**`. This avoids requiring cross-origin browser access to the registry itself.

Behind a reverse proxy, preserve the public origin and protocol headers. The public frontend address is also the registry address displayed in copied pull commands. For remote authenticated use, terminate HTTPS correctly before accepting browser or CLI credentials.

## Compose versus standalone configuration

The bundled Compose file intentionally connects to its own `registry` service with `NUXT_REGISTRY_UPSTREAM=http://registry:5000`. To use another registry in a container deployment, adapt that service environment or run the frontend separately. Do not assume a host-loopback registry is reachable through a container gateway.
## Version-specific configuration

Monitoring is optional and disabled by default. Overview opens the dashboard when enabled; otherwise /home redirects to /explore. Enabling monitoring exposes registry-wide operational data to everyone with UI access, including anonymous visitors of public registries.

| ENV | Default |
| --- | --- |
| `NUXT_MONITORING_ENABLED` | `false` |
| `NUXT_MONITORING_METRICS_URL` | — |
| `NUXT_MONITORING_METRICS_TOKEN` | — |
| `NUXT_MONITORING_WEBHOOK_SECRET` | — |
| `NUXT_MONITORING_DATABASE_PATH` | `.dev/monitoring.sqlite` |

[Monitoring](/docs/v0.4/monitoring)
