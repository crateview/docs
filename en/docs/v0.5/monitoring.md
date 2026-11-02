---
title: "Monitoring"
description: "Monitoring · Distribution · SQLite"
---

## Monitoring

Monitoring is optional and disabled by default. Overview opens the dashboard when enabled; otherwise /home redirects to /explore. Enabling monitoring exposes registry-wide operational data to everyone with UI access, including anonymous visitors of public registries. Managed token mode additionally requires a global read grant.

![Monitoring](https://crateview.de/images/docs/v0.5/monitoring-light.webp)

## Version-specific configuration

Generate a separate webhook secret, then start the overlay below from the matching source checkout. Requires Compose 2.23+. The bundled filesystem configuration must be merged with custom registry authentication or storage settings. Keep debug metrics internal. The overlay mounts persistent SQLite storage; back up its parent directory including WAL files before upgrades.

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

Version-specific configuration. Node.js 26; Distribution; SQLite.

The dashboard offers 24-hour and 7-day windows. Sampling runs every 60 seconds; rates need two samples. Data is retained for 30 days. Restarts establish a new baseline; interruptions, resets and missing metrics produce gaps rather than interpolated history. Notifications have no historical backfill. Manifest events are not complete downloads; storage operation metrics are not disk usage. One server process and one Distribution instance are supported. A 401 probe means reachable with authentication required. During network/5xx outages, an existing verified session may read until expiry; new sessions cannot be authorized. Missing charts: check internal metrics connectivity, the webhook Bearer secret, SQLite write access and two completed samples.
