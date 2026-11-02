---
title: "Installation"
description: "Den kompletten Stack starten oder das UI mit einer vorhandenen Registry verbinden."
---

Beispiele gehören zur Quellrevision dieser Version; für Archive nicht den Standardbranch klonen.


## Voraussetzungen

Verwende Node.js 24.22.2 oder neuer innerhalb von v24 für die lokale Entwicklung oder Docker mit Compose für den kompletten Stack. Eine vorhandene Registry muss Registry HTTP API V2 anbieten. Der mitgelieferte Stack verwendet das offizielle Image `registry:3.1.2`.

## Docker-Compose-Beispiele

Wähle **eine** der folgenden Varianten und speichere sie als `compose.yaml` in einem eigenen Verzeichnis. Ein Repository-Klon ist dafür nicht erforderlich. Erstelle daneben eine `.env`-Datei mit den jeweils angegebenen Werten. Compose übergibt diese ausdrücklich an den Container.

Erzeuge für `NUXT_SESSION_PASSWORD` einen zufälligen Wert mit mindestens 32 Zeichen:

```sh
openssl rand -base64 32
```

Bewahre den Schlüssel dauerhaft auf und schütze `.env`; sie gehört nicht in die Versionsverwaltung. Veröffentlichte Crateview-Images unterstützen Linux amd64.

### UI mit vorhandener Registry

Voraussetzung: eine vom Container erreichbare Registry mit Registry HTTP API V2. Beispiel für `.env` (Adresse ersetzen; keine Zugangsdaten in die URL schreiben):

```dotenv
NUXT_REGISTRY_UPSTREAM=https://registry.example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

```yaml
services:
  frontend:
    image: ghcr.io/crateview/registry-ui:v0.1.2
    ports:
      - "127.0.0.1:3000:3000"
    environment:
      NUXT_REGISTRY_UPSTREAM: ${NUXT_REGISTRY_UPSTREAM:?Set NUXT_REGISTRY_UPSTREAM}
      NUXT_SESSION_PASSWORD: ${NUXT_SESSION_PASSWORD:?Set NUXT_SESSION_PASSWORD}
    restart: unless-stopped
```

```sh
docker compose config --quiet
docker compose pull
docker compose up -d
```

Öffne **http://localhost:3000**. `127.0.0.1` im Upstream bezeichnet den Container selbst. Verwende einen erreichbaren DNS-Namen oder eine Service-Adresse in einem gemeinsamen Docker-Netz.

### UI mit eigener Registry

Lokaler Stack ohne Authentifizierung. Nur der UI-Port wird auf dem Loopback-Interface veröffentlicht; die Registry bleibt im Docker-Netz. `.env`:

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
    image: ghcr.io/crateview/registry-ui:v0.1.2
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
docker compose pull
docker compose up -d
```

Öffne **http://localhost:3000**. Registry-Daten bleiben im Volume `registry-data` erhalten. Ein Image über denselben Origin hochladen und laden:

```sh
docker pull alpine:latest
docker tag alpine:latest localhost:3000/alpine:latest
docker push localhost:3000/alpine:latest
docker pull localhost:3000/alpine:latest
```

### UI, Registry und Traefik mit HTTPS

Eigenständiger Stack einschließlich Traefik und Let's Encrypt. Voraussetzung: ein öffentlich erreichbarer Server, freie und freigegebene TCP-Ports 80/443 sowie eine Domain, deren DNS-Einträge auf diesen Server zeigen. Port 80 muss für die HTTP-01-Challenge erreichbar bleiben. `.env`:

```dotenv
CRATEVIEW_DOMAIN=registry.example.com
ACME_EMAIL=admin@example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

Domain ohne Schema oder Pfad eintragen. Ersetze alle Beispielwerte. Erzeuge anschließend interaktiv eine bcrypt-`htpasswd`-Datei; `your-user` durch den gewünschten Benutzernamen ersetzen:

```sh
mkdir -p auth
chmod 700 auth
docker run --rm -it --mount type=bind,src="$(pwd)/auth",dst=/auth \
  --entrypoint htpasswd httpd:2 -Bc /auth/htpasswd your-user
```

Das Passwort wird abgefragt und erscheint nicht im Befehl. `-c` erstellt die Datei neu; für weitere Benutzer weglassen. Schütze und sichere das Verzeichnis `auth`. Speichere folgende `compose.yaml` daneben:

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
    image: ghcr.io/crateview/registry-ui:v0.1.2
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
docker compose pull
docker compose up -d
```

Öffne **https://registry.example.com** mit deiner Domain und melde dich mit dem erzeugten Registry-Benutzer an. Traefik beschafft das Zertifikat automatisch und leitet HTTP auf HTTPS um. Zertifikate bleiben im Volume `letsencrypt`, Registry-Daten in `registry-data` erhalten. Sichere beide Volumes und verwende beim normalen Stoppen kein `docker compose down -v`.

Alle Pfade einschließlich `/v2/` laufen über Crateview. Registry und UI veröffentlichen keine eigenen Host-Ports. Die Registry prüft Basic Auth; eine zusätzliche Traefik-Basic-Auth ist nicht erforderlich. Traefik erhält den öffentlichen Host und übermittelt das HTTPS-Protokoll an Crateview. Der Docker-Socket erlaubt Traefik Zugriff auf die Docker-API; betreibe den Stack auf einem vertrauenswürdigen Host.

CLI-Anmeldung und Image-Transfer (Domain ersetzen):

```sh
docker login registry.example.com --username your-user
docker pull alpine:latest
docker tag alpine:latest registry.example.com/alpine:latest
docker push registry.example.com/alpine:latest
docker pull registry.example.com/alpine:latest
```

Die CLI fragt das Passwort getrennt von der UI-Anmeldung ab. Weitere Details: [Authentifizierung](/de/docs/v0.1/authentication) und [Traefik mit Docker und Let’s Encrypt](https://doc.traefik.io/traefik/setup/docker/#tls-certificate-management-lets-encrypt).

## Vorhandene Registry verwenden

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout v0.1.2
npm ci
cp .env.example .env
```

Trage den Upstream in `.env` ein und starte den Entwicklungsmodus:

```dotenv
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000
```

```sh
npm run dev
```

Das UI läuft auf Port 3000 mit Hot Module Replacement. Läuft das Frontend in einem Container, bezeichnet `127.0.0.1` diesen Container. Verwende stattdessen eine erreichbare Service-Adresse wie `http://registry:5000`.

## Node-Server für den Produktivbetrieb

```sh
npm run build
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000 HOST=127.0.0.1 PORT=3000 npm start
```

Der produktive Node-Prozess liest tatsächliche Umgebungsvariablen; `.env` wird nicht automatisch eingelesen. Konfiguriere deinen Prozessmanager oder die Container-Umgebung entsprechend. Verwende HTTPS für authentifizierten Zugriff über das Netzwerk.

Crateview benötigt seinen Nitro-Server für Authentifizierung und API-Proxy. Ein statischer Webserver mit den Frontend-Dateien allein liefert kein funktionsfähiges Registry-UI.

Prüfe die [Konfiguration](/de/docs/v0.1/configuration), bevor du den Dienst außerhalb deines eigenen Rechners verfügbar machst.

## Aus Quellcode bauen

Für einen lokalen Build verwende die Compose-Dateien aus dem Quellrepository:

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout v0.1.2
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```
