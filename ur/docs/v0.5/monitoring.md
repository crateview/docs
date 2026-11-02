---
title: "نگرانی"
description: "نگرانی · Distribution · SQLite"
---

## نگرانی

نگرانی اختیاری، بطور طے شدہ بند ہے۔ فعال ہونے پر جائزہ ڈیش بورڈ کھولتا ہے؛ ورنہ /home سے /explore جاتا ہے۔ فعال کرنے سے عوامی رجسٹری کے گمنام زائرین سمیت ہر UI صارف کو پوری رجسٹری کا عملی ڈیٹا ملتا ہے۔ منظم ٹوکن موڈ کو عالمی پڑھنے کی اجازت بھی چاہیے۔

![نگرانی](https://crateview.de/images/docs/v0.5/monitoring-light.webp)

## اس ورژن کی ترتیب

علیحدہ Webhook راز بنائیں اور متعلقہ سورس سے نیچے اوورلے چلائیں۔ Compose 2.23+ چاہیے۔ دی گئی فائل سسٹم ترتیب اپنی تصدیق یا اسٹوریج کے ساتھ ملائیں۔ ڈیبگ میٹرکس اندرونی رکھیں۔ SQLite مستقل ہے؛ اپ گریڈ سے پہلے بنیادی فولڈر اور WAL فائلیں محفوظ کریں۔

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

اس ورژن کی ترتیب. Node.js 26; Distribution; SQLite.

24 گھنٹے اور7 دن کے منظر؛ ہر60 سیکنڈ نمونہ؛ شرح کے لیے دو نمونے۔ ڈیٹا30 دن محفوظ۔ دوبارہ آغاز، کاؤنٹر ری سیٹ اور غائب میٹرکس اندازے کے بغیر خلا بناتے ہیں۔ اطلاعات سابقہ تاریخ نہیں بھرتیں۔ مینی فیسٹ واقعات مکمل ڈاؤن لوڈ نہیں؛ اسٹوریج آپریشن ڈسک استعمال نہیں۔ ایک عمل اور ایک Distribution دستیاب ہے۔401 جانچ کا مطلب رسائی ممکن مگر تصدیق ضروری۔ نیٹ ورک/5xx خرابی میں تصدیق شدہ نشست میعاد تک پڑھ سکتی ہے، نئی مجاز نہیں بنتی۔ چارٹ نہ ہوں تو اندرونی میٹرکس، Webhook Bearer راز، SQLite تحریری اجازت اور دو مکمل نمونے دیکھیں۔
