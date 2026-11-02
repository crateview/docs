---
title: "Monitoring"
description: "Monitoring · Distribution · SQLite"
---

## Monitoring

Monitoring ist optional und standardmäßig deaktiviert. Aktiviert öffnet Übersicht das Dashboard; sonst leitet /home auf /explore weiter. Aktivierung gibt allen Nutzern mit UI-Zugriff Registry-weite Betriebsdaten frei, bei öffentlichen Registries auch anonymen Besuchern. Verwalteter Token-Modus erfordert zusätzlich einen globalen Lese-Grant.

![Monitoring](https://crateview.de/images/docs/next/monitoring-light.webp)

## Versionsspezifische Konfiguration

Erzeuge ein separates Webhook-Geheimnis und starte das folgende Overlay aus dem passenden Quellstand. Compose 2.23+ erforderlich. Die mitgelieferte Dateisystemkonfiguration muss mit eigener Registry-Authentifizierung oder Storage-Konfiguration zusammengeführt werden. Debug-Metriken intern halten. Das Overlay bindet persistenten SQLite-Speicher ein; vor Upgrades Elternverzeichnis inklusive WAL-Dateien sichern.

```sh
openssl rand -hex 32
```

```dotenv
NUXT_MONITORING_WEBHOOK_SECRET=REPLACE_WITH_GENERATED_WEBHOOK_SECRET
```

```sh
docker compose -f compose.yaml -f compose.monitoring.yaml -f compose.build.yaml up --build -d
```

| ENV | Default |
| --- | --- |
| `NUXT_MONITORING_ENABLED` | `false` |
| `NUXT_MONITORING_METRICS_URL` | — |
| `NUXT_MONITORING_METRICS_TOKEN` | — |
| `NUXT_MONITORING_WEBHOOK_SECRET` | — |
| `NUXT_MONITORING_DATABASE_PATH` | `.dev/monitoring.sqlite` |

```yaml
# Distribution: merge with your existing configuration.
http:
  addr: :5000
  debug:
    addr: :5001
    prometheus:
      enabled: true
notifications:
  endpoints:
    - name: crateview
      url: http://frontend:3000/api/monitoring/events
      headers:
        Authorization: ["Bearer REPLACE_WITH_GENERATED_WEBHOOK_SECRET"]
      timeout: 5s
      threshold: 5
      backoff: 1s
```

```dotenv
# Existing Distribution: server-only configuration, internal metrics endpoint.
NUXT_MONITORING_ENABLED=true
NUXT_MONITORING_METRICS_URL=http://registry:5001/metrics
NUXT_MONITORING_METRICS_TOKEN=
NUXT_MONITORING_WEBHOOK_SECRET=REPLACE_WITH_GENERATED_WEBHOOK_SECRET
NUXT_MONITORING_DATABASE_PATH=/data/monitoring.sqlite
```

## 24h / 7d

Versionsspezifische Konfiguration. Node.js 26; Distribution; SQLite.

Das Dashboard bietet 24 Stunden und 7 Tage. Messungen laufen alle 60 Sekunden; Raten benötigen zwei Messpunkte. Aufbewahrung: 30 Tage. Neustarts setzen eine neue Basis; Unterbrechungen, Zähler-Resets und fehlende Metriken erzeugen Lücken. Notifications liefern keine rückwirkende Historie. Manifest-Ereignisse sind keine vollständigen Downloads; Storage-Operationen messen keine Festplattenbelegung. Unterstützt: ein Serverprozess und eine Distribution-Instanz. Eine 401-Probe bedeutet erreichbar mit Anmeldepflicht. Bei Netzwerk-/5xx-Ausfällen dürfen bereits verifizierte Sitzungen bis zum Ablauf lesen; neue Sitzungen werden nicht autorisiert. Fehlende Charts: interne Metriken, Webhook-Bearer-Geheimnis, SQLite-Schreibrechte und zwei abgeschlossene Messungen prüfen.
