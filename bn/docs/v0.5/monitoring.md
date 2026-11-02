---
title: "পর্যবেক্ষণ"
description: "পর্যবেক্ষণ · Distribution · SQLite"
---

## পর্যবেক্ষণ

পর্যবেক্ষণ ঐচ্ছিক, সাধারণত বন্ধ। চালু থাকলে সারসংক্ষেপ ড্যাশবোর্ড; নইলে /home থেকে /explore যায়। চালু করলে পাবলিক রেজিস্ট্রির বেনামি দর্শকসহ সব UI ব্যবহারকারী পুরো রেজিস্ট্রির পরিচালন তথ্য পায়। পরিচালিত টোকেন মোডে বৈশ্বিক পড়ার অনুমতিও লাগে।

![পর্যবেক্ষণ](https://crateview.de/images/docs/v0.5/monitoring-light.webp)

## এই সংস্করণের কনফিগারেশন

আলাদা Webhook গোপন মান তৈরি করে সঠিক সোর্স থেকে নিচের ওভারলে চালান। Compose 2.23+ দরকার। ফাইলসিস্টেম কনফিগারেশন নিজস্ব প্রমাণীকরণ বা স্টোরেজের সঙ্গে মেলান। ডিবাগ মেট্রিক অভ্যন্তরীণ রাখুন। SQLite স্থায়ী; আপগ্রেডের আগে মূল ডিরেক্টরি ও WAL ফাইল ব্যাকআপ নিন।

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

এই সংস্করণের কনফিগারেশন. Node.js 26; Distribution; SQLite.

24 ঘণ্টা ও7 দিনের দৃশ্য; প্রতি60 সেকেন্ডে নমুনা; হার পেতে দুটি লাগে। সংরক্ষণ30 দিন। পুনরারম্ভ, কাউন্টার রিসেট ও অনুপস্থিত মেট্রিকে অনুমান ছাড়া ফাঁক থাকে। বিজ্ঞপ্তি পুরোনো ইতিহাস পূরণ করে না। ম্যানিফেস্ট ঘটনা সম্পূর্ণ ডাউনলোড নয়; স্টোরেজ অপারেশন ডিস্ক ব্যবহার নয়। একটি প্রক্রিয়া ও একটি Distribution সমর্থিত।401 মানে পৌঁছানো যায় কিন্তু প্রমাণীকরণ দরকার। নেটওয়ার্ক/5xx ত্রুটিতে যাচাইকৃত সেশন মেয়াদ পর্যন্ত পড়তে পারে, নতুন অনুমোদন হয় না। চার্ট না থাকলে অভ্যন্তরীণ মেট্রিক, Webhook Bearer গোপন মান, SQLite লেখার অনুমতি ও দুটি সম্পূর্ণ নমুনা দেখুন।
