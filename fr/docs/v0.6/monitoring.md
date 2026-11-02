---
title: "Supervision"
description: "Supervision · Distribution · SQLite"
---

## Supervision

La supervision est facultative et désactivée par défaut. La vue d'ensemble affiche le tableau de bord si elle est active ; sinon /home redirige vers /explore. L'activer expose les données globales à toute personne ayant accès à l'interface, y compris les visiteurs anonymes des registres publics. Le mode à jetons géré exige aussi un droit de lecture global.

![Supervision](https://crateview.de/images/docs/v0.6/monitoring-light.webp)

## Configuration de cette version

Générez un secret Webhook distinct puis lancez la surcharge depuis la révision correspondante. Compose 2.23+ requis. Fusionnez sa configuration de stockage local avec vos réglages personnalisés d'authentification ou de stockage. Gardez les métriques de débogage internes. SQLite est persistant ; sauvegardez son répertoire parent et les fichiers WAL avant mise à jour.

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

Configuration de cette version. Node.js 26; Distribution; SQLite.

Fenêtres de 24 heures et 7 jours, échantillonnage toutes les 60 secondes, deux mesures nécessaires aux taux. Conservation de 30 jours. Redémarrages, compteurs réinitialisés et métriques absentes créent des lacunes sans interpolation. Les notifications ne reconstituent pas le passé. Un événement de manifeste n'est pas un téléchargement complet ; une opération de stockage n'indique pas l'espace disque. Un processus et une instance Distribution sont pris en charge. Une sonde 401 signifie joignable avec authentification requise. Pendant une panne réseau/5xx, les sessions déjà vérifiées lisent jusqu'à expiration ; aucune nouvelle session n'est autorisée. Sans graphiques, vérifiez métriques internes, secret Bearer Webhook, écriture SQLite et deux mesures complètes.
