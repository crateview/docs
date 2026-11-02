---
title: "निगरानी"
description: "निगरानी · Distribution · SQLite"
---

## निगरानी

निगरानी वैकल्पिक है और डिफ़ॉल्ट बंद है। चालू होने पर अवलोकन डैशबोर्ड है; अन्यथा /home से /explore पर भेजा जाता है। चालू करने से सार्वजनिक रजिस्ट्री के अनाम आगंतुकों सहित सभी UI उपयोगकर्ताओं को पूरे रजिस्ट्री का परिचालन डेटा मिलता है। प्रबंधित टोकन मोड को वैश्विक पढ़ने का अधिकार भी चाहिए।

![निगरानी](https://crateview.de/images/docs/next/monitoring-light.webp)

## इस संस्करण का कॉन्फ़िगरेशन

अलग Webhook रहस्य बनाएँ और संबंधित स्रोत से नीचे का ओवरले चलाएँ। Compose 2.23+ चाहिए। दिए फ़ाइलसिस्टम कॉन्फ़िगरेशन को निजी प्रमाणीकरण या स्टोरेज के साथ मिलाएँ। डिबग मेट्रिक्स आंतरिक रखें। SQLite स्थायी है; उन्नयन से पहले मूल फ़ोल्डर और WAL फ़ाइलें सुरक्षित करें।

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

इस संस्करण का कॉन्फ़िगरेशन. Node.js 26; Distribution; SQLite.

24 घंटे और7 दिन के दृश्य; हर60 सेकंड नमूना; दर के लिए दो नमूने। डेटा30 दिन रहता है। पुनःआरंभ, काउंटर रीसेट और अनुपस्थित मेट्रिक्स बिना अनुमान के अंतराल बनाते हैं। सूचनाएँ पुराना इतिहास नहीं भरतीं। मैनिफ़ेस्ट घटनाएँ पूर्ण डाउनलोड नहीं; स्टोरेज ऑपरेशन डिस्क उपयोग नहीं हैं। एक प्रक्रिया और एक Distribution समर्थित हैं।401 जाँच का अर्थ पहुँच योग्य पर प्रमाणीकरण आवश्यक है। नेटवर्क/5xx विफलता में सत्यापित सत्र समाप्ति तक पढ़ते हैं, नए अधिकृत नहीं होते। चार्ट न हों तो आंतरिक मेट्रिक्स, Webhook Bearer रहस्य, SQLite लेखन और दो पूरे नमूने जाँचें।
