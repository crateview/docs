---
title: "Installation"
description: "Start the complete stack or connect the UI to an existing registry."
---

Match examples to this version's source revision; do not clone the default branch for an archived installation.




## Requirements

Use Node.js 26.13.0 or newer within v26 for local development, or Docker with Compose for the complete stack. An existing registry must expose Registry HTTP API V2; the bundled stack uses the official `registry:3.1.2` image.

## Docker Compose examples

Choose **one** of the following variants and save it as `compose.yaml` in its own directory. No repository clone is required. Create a `.env` file alongside it with the values shown for that variant. Compose explicitly passes these values to the container.

Generate a random value of at least 32 characters for `NUXT_SESSION_PASSWORD`:

```sh
openssl rand -base64 32
```

Keep this key stable and protect `.env`; do not commit it to version control. Published Crateview images support Linux amd64.

### UI with an existing registry

Requires a Registry HTTP API V2 endpoint reachable from the container. Example `.env` (replace the address; do not include credentials in the URL):

```dotenv
NUXT_REGISTRY_UPSTREAM=https://registry.example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

```yaml
services:
  frontend:
    image: ghcr.io/crateview/registry-ui:${CRATEVIEW_VERSION:-v0.8.0}
    ports:
      - "127.0.0.1:3000:3000"
    environment:
      NUXT_REGISTRY_UPSTREAM: ${NUXT_REGISTRY_UPSTREAM:?Set NUXT_REGISTRY_UPSTREAM}
      NUXT_SESSION_PASSWORD: ${NUXT_SESSION_PASSWORD:?Set NUXT_SESSION_PASSWORD}
    restart: unless-stopped
```

```sh
docker compose config --quiet
docker compose up -d
```

Open **http://localhost:3000**. An upstream of `127.0.0.1` refers to the container itself. Use a reachable DNS name or a service address on a shared Docker network.

### UI with its own registry

Local stack without authentication. Only the UI port is published on loopback; the registry stays on the Docker network. `.env`:

```dotenv
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

```yaml
services:
  registry:
    image: registry:3.1.2
    environment:
      OTEL_TRACES_EXPORTER: none
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
    volumes:
      - registry-data:/var/lib/registry
    restart: unless-stopped
  frontend:
    image: ghcr.io/crateview/registry-ui:${CRATEVIEW_VERSION:-v0.8.0}
    ports:
      - "127.0.0.1:3000:3000"
    environment:
      NUXT_REGISTRY_UPSTREAM: http://registry:5000
      NUXT_SESSION_PASSWORD: ${NUXT_SESSION_PASSWORD:?Set NUXT_SESSION_PASSWORD}
    depends_on:
      - registry
    restart: unless-stopped

volumes:
  registry-data:
```

```sh
docker compose config --quiet
docker compose up -d
```

Open **http://localhost:3000**. Registry data persists in the `registry-data` volume. Push and pull an image through the same origin:

```sh
docker pull alpine:latest
docker tag alpine:latest localhost:3000/alpine:latest
docker push localhost:3000/alpine:latest
docker pull localhost:3000/alpine:latest
```

### UI, registry and Traefik with HTTPS

Standalone stack including Traefik and Let's Encrypt. Requires a publicly reachable server, available and permitted TCP ports 80/443, and a domain whose DNS records point to that server. Port 80 must remain reachable for the HTTP-01 challenge. `.env`:

```dotenv
CRATEVIEW_DOMAIN=registry.example.com
ACME_EMAIL=admin@example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

Enter the domain without a scheme or path. Replace all example values. Then interactively create a bcrypt `htpasswd` file; replace `your-user` with your username:

```sh
mkdir -p auth
chmod 700 auth
docker run --rm -it --mount type=bind,src="$(pwd)/auth",dst=/auth \
  --entrypoint htpasswd httpd:2 -Bc /auth/htpasswd your-user
```

The password is prompted for and does not appear in the command. `-c` creates a new file; omit it when adding users. Protect and back up the `auth` directory. Save this `compose.yaml` alongside it:

```yaml
services:
  traefik:
    image: traefik:v3.7.17
    command:
      - --providers.docker=true
      - --providers.docker.exposedbydefault=false
      - --entrypoints.web.address=:80
      - --entrypoints.web.http.redirections.entrypoint.to=websecure
      - --entrypoints.web.http.redirections.entrypoint.scheme=https
      - --entrypoints.websecure.address=:443
      - --certificatesresolvers.letsencrypt.acme.email=${ACME_EMAIL:?Set ACME_EMAIL}
      - --certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json
      - --certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - letsencrypt:/letsencrypt
    restart: unless-stopped
  registry:
    image: registry:3.1.2
    environment:
      OTEL_TRACES_EXPORTER: none
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
    volumes:
      - registry-data:/var/lib/registry
      - ./auth:/auth:ro
    restart: unless-stopped
  frontend:
    image: ghcr.io/crateview/registry-ui:${CRATEVIEW_VERSION:-v0.8.0}
    environment:
      NUXT_REGISTRY_UPSTREAM: http://registry:5000
      NUXT_SESSION_PASSWORD: ${NUXT_SESSION_PASSWORD:?Set NUXT_SESSION_PASSWORD}
    labels:
      traefik.enable: "true"
      traefik.http.routers.crateview.rule: Host(`${CRATEVIEW_DOMAIN:?Set CRATEVIEW_DOMAIN}`)
      traefik.http.routers.crateview.entrypoints: websecure
      traefik.http.routers.crateview.tls.certresolver: letsencrypt
      traefik.http.routers.crateview.service: crateview
      traefik.http.services.crateview.loadbalancer.server.port: "3000"
    depends_on:
      - registry
    restart: unless-stopped

volumes:
  registry-data:
  letsencrypt:
```

```sh
docker compose config --quiet
docker compose up -d
```

Open **https://registry.example.com** using your domain and sign in with the registry user you created. Traefik obtains the certificate automatically and redirects HTTP to HTTPS. Certificates persist in `letsencrypt`, registry data in `registry-data`. Back up both volumes and do not use `docker compose down -v` for routine shutdowns.

All paths, including `/v2/`, go through Crateview. Neither the registry nor the UI publishes its own host ports. The registry enforces Basic Auth; no additional Traefik Basic Auth is required. Traefik preserves the public host and forwards the HTTPS protocol to Crateview. The Docker socket gives Traefik access to the Docker API; run this stack on a trusted host.

CLI login and image transfer (replace the domain):

```sh
docker login registry.example.com --username your-user
docker pull alpine:latest
docker tag alpine:latest registry.example.com/alpine:latest
docker push registry.example.com/alpine:latest
docker pull registry.example.com/alpine:latest
```

The CLI prompts for the password separately from the UI login. See [authentication](/docs/v0.8/authentication) and [Traefik with Docker and Let’s Encrypt](https://doc.traefik.io/traefik/setup/docker/#tls-certificate-management-lets-encrypt) for details.

## Use an existing registry

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 244ad8eb2cea0317f9ab149c238def5b414af565
npm ci
cp .env.example .env
```

Set the upstream in `.env`, then start development mode:

```dotenv
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000
```

```sh
npm run dev
```

The UI runs on port 3000 with hot module replacement. When the frontend runs inside a container, `127.0.0.1` means that container. Use a reachable service address, such as `http://registry:5000`, instead.

## Production Node server

```sh
npm run build
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000 HOST=127.0.0.1 PORT=3000 npm start
```

The production Node process reads actual environment variables, not `.env` automatically. Configure your process manager or container environment accordingly. Serve HTTPS for remote authenticated use.

Crateview needs its Nitro server for authentication and API proxying. Serving only the frontend files as a static website will not provide a working registry UI.

Continue with [configuration](/docs/v0.8/configuration) before exposing the service beyond your own machine.

## Build from source

For a local build, use the Compose files from the source repository:

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 244ad8eb2cea0317f9ab149c238def5b414af565
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```

<a id="pwa"></a>

## Install as an app (PWA)

Install Crateview through the native install or Add to Home Screen action of a supported browser. It opens in standalone mode. The application still needs its Nitro server and a reachable registry.

Use HTTPS in production; localhost is suitable for local testing. The service worker is enabled only in production builds, not during `npm run dev`. Build and start the production server before testing installation or offline behavior.

After a successful online visit and service-worker installation, opening or reloading the app without a network connection shows a connection message. Retry reloads the original URL after connectivity returns. The saved language choice takes precedence over browser languages, with English fallback; Arabic and Urdu use RTL.

Only app document navigations use the offline fallback. Registry, account and session responses are not saved to Cache Storage; there is no offline registry browsing. Existing open screens retain their normal request-error behavior. API and `/v2/` traffic is not replaced with offline HTML, and worker updates do not force open tabs to reload.
