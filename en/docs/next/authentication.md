---
title: 'Authentication'
description: 'Anonymous access, Basic Auth, encrypted sessions and HTTPS.'
---

## Supported authentication

Crateview supports anonymous registries, native htpasswd Basic Auth and the optional Crateview auth service in token mode. It probes `/v2/` and displays the sign-in form when the registry requires Basic authentication.

External Bearer-token services and OIDC are not integrated. Unconfigured external token services produce an explicit unsupported-auth notice.

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

## Optional account management

An independent auth/management service supports account creation and own-password changes. Docker connects directly to the registry and works without the UI container. Users and bcrypt hashes stay in `htpasswd`; token permissions live in `permissions.json`. Account storage does not use SQLite.

- **Basic mode:** Distribution checks the shared htpasswd file; all users have equal registry access. Docker does not require the management service to run.
- **Token mode:** The independent auth service issues signed five-minute tokens; Distribution enforces repository permissions. Docker requires the registry and auth service. New users initially have no repository access.
- **UI:** The server allowlist controls listing, creation, password resets and account deletion. `NUXT_USER_CREATION_ALLOWED_USERS`: `alice,bob`; `*`. Empty denies administration; listed names are allowed, and the wildcard allows every authenticated user.
- **Permissions:** CLI grants target exact repositories, namespaces (`team/*`) or everything (`*`); profiles are `reader` (pull), `writer` (pull/push), and `manager` (pull/push/delete). Basic mode has no registry roles.

Explicitly enable `NUXT_AUTH_MODE=htpasswd` or `token` and configure the internal `NUXT_AUTH_SERVICE_URL` and server-only `NUXT_AUTH_SERVICE_SECRET`. Token mode additionally requires `NUXT_AUTH_TOKEN_REALM` to match the public token endpoint exactly. Anonymous registries and external token services do not support local account management. Repository grants are managed through the auth CLI; the UI displays them read-only.

See the [account service guide](https://github.com/crateview/registry-ui/blob/3844d7605dac80d0d58999201c43e94c644d72c0/auth/README.md) for Compose overlays, setup, migration and CLI commands. Route public Docker access directly to the registry independently of the UI. Previously issued tokens remain valid for up to five minutes after password changes or permission revocation.

Administration groups Settings and Users. The account menu also offers language and theme selection. Account changes your own password after confirming the current password. Users lists accounts and creates them only when the server allowlist permits it; token grants are read-only. New accounts require a password of at least 12 characters and at most 72 UTF-8 bytes. New token accounts start without repository access.

The server allowlist controls listing, creation, password resets and account deletion. The right-aligned three-dot menu lets administrators reset another user's password without the old password or delete that account after confirmation. Resets preserve repository grants; deletion removes them. Your own menu opens Account for a password change with the current password; self-deletion is blocked.

![Documentation version](https://crateview.de/images/docs/next/users-light.webp)

![Actions](https://crateview.de/images/docs/next/user-actions-light.webp)

![Change password](https://crateview.de/images/docs/next/user-password-light.webp)

![Delete user](https://crateview.de/images/docs/next/user-delete-light.webp)

```dotenv
NUXT_AUTH_MODE=htpasswd
NUXT_AUTH_SERVICE_URL=http://auth:8080
NUXT_AUTH_SERVICE_SECRET=REPLACE_WITH_SEPARATE_MANAGEMENT_SECRET
NUXT_USER_CREATION_ALLOWED_USERS=alice,bob
```

![Documentation version](https://crateview.de/images/docs/next/account-light.webp)

![Documentation version](https://crateview.de/images/docs/next/login-light.webp)
