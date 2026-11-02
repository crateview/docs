---
title: "Мониторинг"
description: "Мониторинг · Distribution · SQLite"
---

## Мониторинг

Мониторинг необязателен и изначально выключен. При включении Обзор открывает панель, иначе /home перенаправляет на /explore. Включение раскрывает общие эксплуатационные данные всем пользователям UI, включая анонимных посетителей публичных реестров. Управляемый токенный режим дополнительно требует глобального права чтения.

![Мониторинг](https://crateview.de/images/docs/v0.7/monitoring-light.webp)

## Настройки этой версии

Создайте отдельный секрет Webhook и запустите overlay из соответствующей ревизии. Нужен Compose 2.23+. Объедините файловую конфигурацию с собственной аутентификацией или хранилищем. Метрики отладки оставляйте внутренними. SQLite сохраняется; перед обновлением копируйте родительский каталог вместе с WAL.

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

Настройки этой версии. Node.js 26; Distribution; SQLite.

Окна 24 часа и 7 дней, измерения каждые 60 секунд; для скоростей нужны две точки. Хранение 30 дней. Перезапуски, сбросы счётчиков и пропуски метрик создают разрывы без интерполяции. Уведомления не восстанавливают прошлое. События манифестов не равны полным загрузкам; операции хранилища не показывают занятое место. Поддерживаются один процесс и одна Distribution. Ответ пробы 401 означает доступность с обязательной аутентификацией. При сетевых/5xx-сбоях проверенные сессии читают до истечения срока; новые не авторизуются. Если нет графиков, проверьте внутренние метрики, Bearer-секрет Webhook, запись SQLite и две завершённые выборки.
