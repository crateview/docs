---
title: 'Authentication'
description: 'Anonymous access, Basic Auth, encrypted sessions and HTTPS.'
---

## Supported authentication

Crateview supports anonymous registries and native htpasswd Basic Auth. It probes `/v2/` and displays the sign-in form when the registry requires Basic authentication.

External Bearer-token services and OIDC are not integrated. A token challenge produces an explicit unsupported-auth notice rather than a misleading login form.

## Registry-side Basic Auth

Create an htpasswd file with a bcrypt password hash. The following example uses a temporary official Apache container; replace the example credentials and protect the resulting file:

```sh
mkdir -p auth
docker run --rm --entrypoint htpasswd httpd:2   -Bbn your-user choose-a-strong-password > auth/htpasswd
```

Example Distribution settings:

```yaml
auth:
  htpasswd:
    realm: Registry
    path: /auth/htpasswd
```

Mount the generated file into the registry container at `/auth/htpasswd`. Registry configuration is separate from Crateview configuration. Use HTTPS for remote access; Basic Auth is an encoding, not transport encryption. Avoid leaving real passwords in shared shell history or public examples.

## UI sessions

The Nitro server validates credentials against the registry. It stores them in an encrypted, HttpOnly, SameSite=Strict cookie. Browser JavaScript and localStorage do not receive the credentials. Sessions expire after eight hours.

Set a random production session secret:

```sh
openssl rand -base64 32
```

Use the generated value as `NUXT_SESSION_PASSWORD`, with at least 32 characters. Every frontend instance behind a load balancer must use the same value. Without configuration, Crateview generates a process-local secret, so a restart invalidates existing sessions.

## HTTPS and logout

The session cookie uses Secure when the public request protocol is HTTPS. Configure the reverse proxy's forwarded protocol correctly. Signing out clears the UI session; it does not delete or modify registry users.

Docker and other CLI clients authenticate directly through the `/v2/**` proxy using their normal registry credentials. Signing into the web UI does not sign the Docker CLI in automatically.
