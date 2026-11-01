---
title: 'Installation'
description: 'Den kompletten Stack starten oder das UI mit einer vorhandenen Registry verbinden.'
---

## Voraussetzungen

Verwende Node.js 24 für die lokale Entwicklung oder Docker mit Compose für den kompletten Stack. Eine vorhandene Registry muss Registry HTTP API V2 anbieten. Der mitgelieferte Stack verwendet das offizielle Image `registry:3.1.2`.

## Kompletter Docker-Compose-Stack

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
docker compose pull
docker compose up -d
```

Öffne **http://localhost:3000**. Compose lädt das veröffentlichte Image `ghcr.io/crateview/registry-ui:v0.1.0` und startet die Registry in einem internen Docker-Netz. Die Registry-Daten bleiben im Volume `registry-data` erhalten. Standardmäßig wird nur der Frontend-Port auf dem Loopback-Interface veröffentlicht.

Ein Image kannst du über denselben Origin hochladen:

```sh
docker pull alpine:latest
docker tag alpine:latest localhost:3000/alpine:latest
docker push localhost:3000/alpine:latest
```

## Vorhandene Registry verwenden

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
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

Prüfe die [Konfiguration](/de/docs/configuration), bevor du den Dienst außerhalb deines eigenen Rechners verfügbar machst.

## Aus Quellcode bauen

Veröffentlichte Images unterstützen Linux amd64. Für einen lokalen Build:

```sh
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```
