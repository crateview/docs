---
title: "Monitorización"
description: "Monitorización · Distribution · SQLite"
---

## Monitorización

La monitorización es opcional y está desactivada por defecto. Resumen abre el panel si está activada; si no, /home redirige a /explore. Activarla da datos operativos globales a todos con acceso a la UI, incluso visitantes anónimos de registros públicos.

![Monitorización](https://crateview.de/images/docs/v0.4/monitoring-light.webp)

## Configuración de esta versión

Genera un secreto Webhook independiente e inicia la superposición desde la revisión correspondiente. Requiere Compose 2.23+. Integra su configuración de almacenamiento local con autenticación o almacenamiento personalizados. Mantén internas las métricas de depuración. SQLite es persistente; respalda su directorio padre y archivos WAL antes de actualizar.

```sh
openssl rand -hex 32
```

```dotenv
CRATEVIEW_VERSION=v0.4.0
NUXT_MONITORING_WEBHOOK_SECRET=REPLACE_WITH_GENERATED_WEBHOOK_SECRET
```

```sh
docker compose -f compose.yaml -f compose.monitoring.yaml up -d
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

Configuración de esta versión. Node.js 24; Distribution; SQLite.

El panel ofrece 24 horas y 7 días. Muestrea cada 60 segundos; las tasas necesitan dos muestras. Conserva 30 días. Reinicios crean una base nueva; interrupciones, reinicios de contadores y métricas ausentes dejan huecos sin interpolación. No hay relleno histórico de notificaciones. Eventos de manifiesto no son descargas completas; operaciones de almacenamiento no son uso de disco. Se admite un proceso y una instancia Distribution. Una prueba 401 significa accesible con autenticación requerida. Durante fallos de red/5xx, sesiones verificadas pueden leer hasta expirar; no se autorizan sesiones nuevas. Si faltan gráficas, comprueba métricas internas, secreto Bearer Webhook, escritura SQLite y dos muestras completas.
