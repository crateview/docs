---
title: 'Troubleshooting'
description: 'Resolve connection, authentication, metadata and deployment issues.'
---

## The UI cannot reach the registry

Check the registry origin from the frontend server's network environment. In a container, `localhost` points to the container itself. A common Compose upstream is `http://registry:5000`.

```sh
curl -i http://127.0.0.1:5000/v2/
```

An anonymous registry should return success. An authenticated one can return `401` with a `WWW-Authenticate` challenge. A connection refusal means the listener, address or network path is not ready.

## Sign-in fails

Crateview supports anonymous registries, native htpasswd Basic Auth and the optional Crateview auth service in token mode. It probes `/v2/` and displays the sign-in form when the registry requires Basic authentication.

External Bearer-token services and OIDC are not integrated. Unconfigured external token services produce an explicit unsupported-auth notice.

For repeated session loss, check that all frontend instances share `NUXT_SESSION_PASSWORD`. A process-local generated secret changes after a restart. Also verify HTTPS and forwarded protocol headers; browser sessions require the cookie to return to the same origin.

## My image has no description or creation date

These values must exist in the manifest annotations or image config. Crateview does not infer missing metadata or substitute registry push times. Add the appropriate OCI annotations when building your images.

## Referrers are empty

The registry needs to expose the OCI referrers endpoint or the standard digest-tag fallback index. OCI clients are responsible for maintaining that fallback index when required. Merely pushing a subject-bearing manifest into a registry without referrers support does not make Crateview scan all tags for it.

## Deletion is missing or rejected

Enable both Crateview's `NUXT_REGISTRY_DELETE_ENABLED` and Distribution's storage delete setting. The registry can still reject the request according to its permissions or implementation. Deletion works by digest and may affect multiple tags.

## Production ignores my .env file

`node .output/server/index.mjs` does not automatically load `.env`. Export variables through your process manager, container environment or shell. Restart after changing runtime values. A static-only web host cannot replace the required Nitro server.

## The copied pull command does not work

The command uses the public frontend address. Confirm that the CLI can reach that origin and trusts its TLS certificate. Authenticate the CLI separately when needed. A browser sign-in does not create a Docker CLI login.

If you find a reproducible UI issue, open an [issue](https://github.com/crateview/registry-ui/issues) with the affected revision and sanitized reproduction details. Use the [private security channel](https://github.com/crateview/registry-ui/security/policy) for vulnerabilities.
## Monitoring

Version-specific configuration: [Monitoring](/docs/next/monitoring).

Version-specific configuration — Monitoring is optional and disabled by default. Overview opens the dashboard when enabled; otherwise /home redirects to /explore. Enabling monitoring exposes registry-wide operational data to everyone with UI access, including anonymous visitors of public registries. Managed token mode additionally requires a global read grant.

## PWA installation or offline startup does not work

Check browser installation support, HTTPS or localhost, and a production build. Visit the app online first and allow the worker to install. Requests for `manifest.webmanifest` and `sw.js` must reach Crateview under the configured `NUXT_APP_BASE_URL`; a reverse proxy must preserve that prefix and serve the correct manifest and JavaScript responses.

An offline message is expected while the network is unavailable. It does not provide stored repository or account data. Reconnect and retry. For incorrect icons, check the exact PNG dimensions, readable container paths, the file-over-URL precedence and server startup errors; restart after changes. Browsers may refresh installed icons later.
