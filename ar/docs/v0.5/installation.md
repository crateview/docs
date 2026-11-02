---
title: "تثبيت"
description: "ابدأ تشغيل المكدس الكامل أو قم بتوصيل واجهة المستخدم بسجل موجود."
---

استخدم مراجعة هذا الإصدار ولا تستنسخ الفرع الافتراضي لتثبيت مؤرشف.

## متطلبات

استخدم Node.js 26.13.0 أو الأحدث ضمن الإصدار 26 للتطوير المحلي، أو Docker مع Compose للمكدس الكامل. يجب أن يعرض السجل الموجود Registry HTTP API V2؛ تستخدم الحزمة المجمعة صورة `registry:3.1.2` الرسمية.

## أمثلة Docker Compose

اختر **واحدًا** من المتغيرات التالية واحفظه باسم `compose.yaml` في الدليل الخاص به. لا يلزم استنساخ المستودع. قم بإنشاء ملف `.env` بجانبه بالقيم الموضحة لهذا المتغير. يقوم الخيار Compose بتمرير هذه القيم بشكل صريح إلى الحاوية.

قم بإنشاء قيمة عشوائية لا تقل عن 32 حرفًا لـ `NUXT_SESSION_PASSWORD`:

```sh
openssl rand -base64 32
```

حافظ على استقرار هذا المفتاح وحماية `.env`؛ لا تلزمه بالتحكم في الإصدار. تدعم صور Crateview المنشورة Linux AMD64.

### واجهة المستخدم مع التسجيل الموجود

يتطلب نقطة نهاية تسجيل HTTP API V2 يمكن الوصول إليها من الحاوية. مثال `.env` (استبدل العنوان؛ لا تقم بتضمين بيانات الاعتماد في عنوان URL):

```dotenv
NUXT_REGISTRY_UPSTREAM=https://registry.example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

```yaml
services:
  frontend:
    image: ghcr.io/crateview/registry-ui:v0.5.0
    ports:
      - "127.0.0.1:3000:3000"
    environment:
      NUXT_REGISTRY_UPSTREAM: ${NUXT_REGISTRY_UPSTREAM:?Set NUXT_REGISTRY_UPSTREAM}
      NUXT_SESSION_PASSWORD: ${NUXT_SESSION_PASSWORD:?Set NUXT_SESSION_PASSWORD}
    restart: unless-stopped
```

```sh
docker compose config --quiet
docker compose up -d
```

Open **http://localhost:3000**. يشير المنبع `127.0.0.1` إلى الحاوية نفسها. استخدم اسم DNS يمكن الوصول إليه أو عنوان خدمة على شبكة Docker مشتركة.

### واجهة المستخدم مع التسجيل الخاص بها

المكدس المحلي دون مصادقة. يتم نشر منفذ واجهة المستخدم فقط عند الاسترجاع؛ يبقى التسجيل على شبكة Docker. `.env`:

```dotenv
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

```yaml
services:
  registry:
    image: registry:3.1.2
    environment:
      OTEL_TRACES_EXPORTER: none
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
    volumes:
      - registry-data:/var/lib/registry
    restart: unless-stopped
  frontend:
    image: ghcr.io/crateview/registry-ui:v0.5.0
    ports:
      - "127.0.0.1:3000:3000"
    environment:
      NUXT_REGISTRY_UPSTREAM: http://registry:5000
      NUXT_SESSION_PASSWORD: ${NUXT_SESSION_PASSWORD:?Set NUXT_SESSION_PASSWORD}
    depends_on:
      - registry
    restart: unless-stopped

volumes:
  registry-data:
```

```sh
docker compose config --quiet
docker compose up -d
```

فتح ** http://localhost:3000**. تستمر بيانات التسجيل في وحدة تخزين `registry-data`. Push وpull صورة من نفس الأصل:

```sh
docker pull alpine:latest
docker tag alpine:latest localhost:3000/alpine:latest
docker push localhost:3000/alpine:latest
docker pull localhost:3000/alpine:latest
```

### واجهة المستخدم والتسجيل وTraefik مع HTTPS

مكدس مستقل يتضمن Traefik وLet's Encrypt. يتطلب خادمًا يمكن الوصول إليه بشكل عام، ومنافذ TCP متاحة ومسموح بها 80/443، ومجال تشير سجلات DNS الخاصة به إلى ذلك الخادم. يجب أن يظل المنفذ 80 قابلاً للوصول لتحدي HTTP-01. `.env`:

```dotenv
CRATEVIEW_DOMAIN=registry.example.com
ACME_EMAIL=admin@example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

أدخل المجال بدون مخطط أو مسار. استبدل جميع قيم الأمثلة. ثم قم بإنشاء ملف bcrypt `htpasswd` بشكل تفاعلي؛ استبدل `your-user` باسم المستخدم الخاص بك:

```sh
mkdir -p auth
chmod 700 auth
docker run --rm -it --mount type=bind,src="$(pwd)/auth",dst=/auth \
  --entrypoint htpasswd httpd:2 -Bc /auth/htpasswd your-user
```

تتم المطالبة بكلمة المرور ولا تظهر في الأمر. يقوم `-c` بإنشاء ملف جديد؛ حذفه عند إضافة المستخدمين. حماية وعمل نسخة احتياطية من دليل `auth`. احفظ `compose.yaml` بجانبه:

```yaml
services:
  traefik:
    image: traefik:v3.7.17
    command:
      - --providers.docker=true
      - --providers.docker.exposedbydefault=false
      - --entrypoints.web.address=:80
      - --entrypoints.web.http.redirections.entrypoint.to=websecure
      - --entrypoints.web.http.redirections.entrypoint.scheme=https
      - --entrypoints.websecure.address=:443
      - --certificatesresolvers.letsencrypt.acme.email=${ACME_EMAIL:?Set ACME_EMAIL}
      - --certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json
      - --certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - letsencrypt:/letsencrypt
    restart: unless-stopped
  registry:
    image: registry:3.1.2
    environment:
      OTEL_TRACES_EXPORTER: none
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
    volumes:
      - registry-data:/var/lib/registry
      - ./auth:/auth:ro
    restart: unless-stopped
  frontend:
    image: ghcr.io/crateview/registry-ui:v0.5.0
    environment:
      NUXT_REGISTRY_UPSTREAM: http://registry:5000
      NUXT_SESSION_PASSWORD: ${NUXT_SESSION_PASSWORD:?Set NUXT_SESSION_PASSWORD}
    labels:
      traefik.enable: "true"
      traefik.http.routers.crateview.rule: Host(`${CRATEVIEW_DOMAIN:?Set CRATEVIEW_DOMAIN}`)
      traefik.http.routers.crateview.entrypoints: websecure
      traefik.http.routers.crateview.tls.certresolver: letsencrypt
      traefik.http.routers.crateview.service: crateview
      traefik.http.services.crateview.loadbalancer.server.port: "3000"
    depends_on:
      - registry
    restart: unless-stopped

volumes:
  registry-data:
  letsencrypt:
```

```sh
docker compose config --quiet
docker compose up -d
```

افتح **https://registry.example.com** باستخدام المجال الخاص بك وقم بتسجيل الدخول باستخدام مستخدم التسجيل الذي قمت بإنشائه. يحصل Traefik على الشهادة تلقائيًا ويعيد توجيه HTTP إلى HTTPS. تستمر الشهادات في `letsencrypt`، وبيانات التسجيل في `registry-data`. قم بعمل نسخة احتياطية لكلا المجلدين ولا تستخدم `docker compose down -v` لعمليات إيقاف التشغيل الروتينية.

تمر كافة المسارات، بما في ذلك `/v2/`، عبر Crateview. لا يقوم السجل ولا واجهة المستخدم بنشر منافذ المضيف الخاصة بهم. يفرض التسجيل Basic Auth؛ لا يلزم وجود Traefik Basic Auth إضافي. يحتفظ Traefik بالمضيف العام ويعيد توجيه بروتوكول HTTPS إلى Crateview. يمنح مقبس Docker Traefik إمكانية الوصول إلى Docker API؛ قم بتشغيل هذا المكدس على مضيف موثوق به.

تسجيل الدخول إلى CLI ونقل الصور (استبدل المجال):

```sh
docker login registry.example.com --username your-user
docker pull alpine:latest
docker tag alpine:latest registry.example.com/alpine:latest
docker push registry.example.com/alpine:latest
docker pull registry.example.com/alpine:latest
```

يطالب سطر الأوامر بكلمة المرور بشكل منفصل عن تسجيل الدخول إلى واجهة المستخدم. راجع [authentication](/ar/docs/v0.5/authentication) و[Traefik with Docker وLet’s Encrypt](https://doc.traefik.io/traefik/setup/docker/#tls-certificate-management-lets-encrypt) للحصول على التفاصيل.

## استخدام التسجيل الموجود

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout v0.5.0
npm ci
cp .env.example .env
```

قم بتعيين المنبع في `.env`، ثم ابدأ وضع التطوير:

```dotenv
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000
```

```sh
npm run dev
```

تعمل واجهة المستخدم على المنفذ 3000 مع استبدال الوحدة الساخنة. عندما تعمل الواجهة الأمامية داخل حاوية، فإن `127.0.0.1` تعني تلك الحاوية. استخدم عنوان خدمة يمكن الوصول إليه، مثل `http://registry:5000`، بدلاً من ذلك.

## خادم عقدة الإنتاج

```sh
npm run build
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000 HOST=127.0.0.1 PORT=3000 npm start
```

تقرأ عملية عقدة الإنتاج متغيرات البيئة الفعلية، وليس `.env` تلقائيًا. قم بتكوين مدير العمليات أو بيئة الحاوية الخاصة بك وفقًا لذلك. خدمة HTTPS للاستخدام المصادق عليه عن بعد.

يحتاج Crateview إلى خادم Nitro الخاص به للمصادقة والوكيل API. إن عرض ملفات الواجهة الأمامية فقط كموقع ويب ثابت لن يوفر واجهة مستخدم تسجيل فعالة.

تابع مع [configuration](/ar/docs/v0.5/configuration) قبل الكشف عن الخدمة خارج جهازك.

## البناء من المصدر

للبناء المحلي، استخدم ملفات Compose من مستودع المصدر:

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout v0.5.0
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```
