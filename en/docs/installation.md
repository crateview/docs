---
title: 'Installation'
description: 'Start the complete stack or connect the UI to an existing registry.'
---

## Requirements

Use Node.js 24.22.2 or newer within v24 for local development, or Docker with Compose for the complete stack. An existing registry must expose Registry HTTP API V2; the bundled stack uses the official `registry:3.1.2` image.

## Complete Docker Compose stack

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
docker compose pull
docker compose up -d
```

Open **http://localhost:3000**. Compose downloads the released `ghcr.io/crateview/registry-ui:v0.1.0` image and starts the registry on an internal Docker network. Registry storage persists in the `registry-data` volume. Only the frontend port is published, bound to loopback by default.

Push an image through the same origin:

```sh
docker pull alpine:latest
docker tag alpine:latest localhost:3000/alpine:latest
docker push localhost:3000/alpine:latest
```

## Use an existing registry

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
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

Continue with [configuration](/docs/configuration) before exposing the service beyond your own machine.

## Build from source

Published images support Linux amd64. To build locally instead:

```sh
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```
