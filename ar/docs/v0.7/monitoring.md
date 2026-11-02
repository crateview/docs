---
title: "المراقبة"
description: "المراقبة · Distribution · SQLite"
---

## المراقبة

المراقبة اختيارية ومعطلة افتراضياً. تفتح النظرة العامة لوحة المراقبة عند تفعيلها؛ وإلا يحوّل /home إلى /explore. يمنح التفعيل البيانات التشغيلية الشاملة لكل من يصل للواجهة، بما فيهم زوار السجلات العامة المجهولون. يتطلب وضع الرموز المُدار إذن قراءة عام أيضاً.

![المراقبة](https://crateview.de/images/docs/v0.7/monitoring-light.webp)

## إعدادات هذا الإصدار

أنشئ سراً مستقلاً للـ Webhook ثم شغّل التراكب أدناه من المراجعة المطابقة. يتطلب Compose 2.23+. ادمج إعداد نظام الملفات مع مصادقة السجل أو تخزينه المخصص. أبق مقاييس التصحيح داخلية. SQLite دائم؛ انسخ المجلد الأب وملفات WAL قبل الترقية.

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

إعدادات هذا الإصدار. Node.js 26; Distribution; SQLite.

نوافذ24 ساعة و7 أيام، وأخذ عينات كل60 ثانية؛ تحتاج المعدلات عينتين. الاحتفاظ30 يوماً. تنشئ إعادة التشغيل وإعادة ضبط العدادات والمقاييس المفقودة فجوات دون استيفاء. لا تستعيد الإشعارات التاريخ السابق. أحداث البيان ليست تنزيلات كاملة وعمليات التخزين ليست استخدام القرص. يدعم عملية واحدة ومثيل Distribution واحداً. تعني نتيجة401 إمكانية الوصول مع وجوب المصادقة. عند أخطاء الشبكة/5xx تقرأ الجلسات المتحققة حتى انتهائها ولا تُعتمد جلسات جديدة. عند غياب الرسوم افحص المقاييس الداخلية وسر Webhook Bearer وصلاحية كتابة SQLite وعينتين كاملتين.
