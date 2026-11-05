---
title: 'Configuration'
description: 'Registry connection, runtime environment variables and deployment behavior.'
---

## Configuration layers

Application defaults are defined in `nuxt.config.ts` and `app/app.config.ts`. Environment variables override runtime defaults when the server starts. Development and build commands read `.env`; the production Node entrypoint does not read it automatically.

Restart the server after changing runtime environment variables. Editing source-level component configuration requires a new production build.

## Date and time format

Set `NUXT_PUBLIC_DATE_FORMAT` at runtime to format all dates in the interface. The browser supplies the time zone. If the variable is empty or invalid, Crateview uses the browser’s locale.

Example: `DD.MM.YYYY HH:mm`. Tokens: `YYYY`, `MM`, `DD`, `HH`, `mm`, `ss`, `Z` (UTC ±HH:mm). Put literal text in square brackets, for example `[at]`.

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

Public configuration is sent to the browser. Never store passwords or tokens in `NUXT_PUBLIC_*` variables. See [branding and theming](/docs/v0.8/theming) for color and typography options.

## Network layout

The browser communicates with Crateview on one origin. Nitro proxies `/v2/**` for registry clients and reads registry metadata through `/api/registry/**`. This avoids requiring cross-origin browser access to the registry itself.

Behind a reverse proxy, preserve the public origin and protocol headers. The public frontend address is also the registry address displayed in copied pull commands. For remote authenticated use, terminate HTTPS correctly before accepting browser or CLI credentials.

## Compose versus standalone configuration

The bundled Compose file intentionally connects to its own `registry` service with `NUXT_REGISTRY_UPSTREAM=http://registry:5000`. To use another registry in a container deployment, adapt that service environment or run the frontend separately. Do not assume a host-loopback registry is reachable through a container gateway.
## Version-specific configuration

Monitoring is optional and disabled by default. Overview opens the dashboard when enabled; otherwise /home redirects to /explore. Enabling monitoring exposes registry-wide operational data to everyone with UI access, including anonymous visitors of public registries. Managed token mode additionally requires a global read grant.

| ENV | Default |
| --- | --- |
| `NUXT_MONITORING_ENABLED` | `false` |
| `NUXT_MONITORING_METRICS_URL` | — |
| `NUXT_MONITORING_METRICS_TOKEN` | — |
| `NUXT_MONITORING_WEBHOOK_SECRET` | — |
| `NUXT_MONITORING_DATABASE_PATH` | `.dev/monitoring.sqlite` |

[Monitoring](/docs/v0.8/monitoring)

| ENV | Default |
| --- | --- |
| `NUXT_AUTH_MODE` | `off` |
| `NUXT_AUTH_SERVICE_URL` | — |
| `NUXT_AUTH_SERVICE_SECRET` | — |
| `NUXT_AUTH_TOKEN_REALM` | — |
| `NUXT_USER_CREATION_ALLOWED_USERS` | — |

Administration groups Settings and Users. The account menu also offers language and theme selection. Account changes your own password after confirming the current password. Users lists accounts and creates them only when the server allowlist permits it; token grants are read-only. New accounts require a password of at least 12 characters and at most 72 UTF-8 bytes. New token accounts start without repository access.

The server allowlist controls listing, creation, password resets and account deletion. The right-aligned three-dot menu lets administrators reset another user's password without the old password or delete that account after confirmation. Resets preserve repository grants; deletion removes them. Your own menu opens Account for a password change with the current password; self-deletion is blocked.

## PWA icon configuration

| Variable | Default | Purpose |
| --- | --- | --- |
| `NUXT_PUBLIC_BRAND_PWA_ICON_192` | `/branding/pwa-192.png` | 192-pixel install icon URL. |
| `NUXT_PUBLIC_BRAND_PWA_ICON_512` | `/branding/pwa-512.png` | 512-pixel install icon URL. |
| `NUXT_BRAND_PWA_ICON_192_FILE` | — | Private 192-pixel PNG file override. |
| `NUXT_BRAND_PWA_ICON_512_FILE` | — | Private 512-pixel PNG file override. |

Local icon files override their corresponding public URLs. They must be readable, nonempty PNGs with the exact square dimensions. Invalid files prevent server startup. Relative file paths resolve against the server working directory; container paths must refer to readable files inside the container. Restart after changing configuration.

Local icon URLs and the manifest respect `NUXT_APP_BASE_URL`, for example `/registry/`; external HTTP(S) icon URLs are preserved. Never use the private file variables for secrets: their image bytes are public, although their filesystem paths remain server-side.
