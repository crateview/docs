---
title: "تنصیب"
description: "مکمل اسٹیک شروع کریں یا UI کو موجودہ رجسٹری سے جوڑیں۔"
---

اس ورژن کا سورس استعمال کریں؛ محفوظ تنصیب کے لیے طے شدہ شاخ کلون نہ کریں۔

پیش نظارہ کنٹینر پہلے مقررہ سورس سے مقامی بنائیں؛ شائع شدہ امیج فرض نہ کریں۔

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 3844d7605dac80d0d58999201c43e94c644d72c0
docker build -t crateview-registry-ui:next .
```


## تقاضے

مقامی ترقی کے لیے V26 کے اندر Node.js 26.13.0 یا جدید تر استعمال کریں، یا مکمل اسٹیک کے لیے Compose کے ساتھ Docker استعمال کریں۔ ایک موجودہ رجسٹری کو رجسٹری HTTP API V2 کو بے نقاب کرنا چاہیے؛ بنڈل اسٹیک سرکاری `registry:3.1.2` تصویر کا استعمال کرتا ہے۔

## Docker Compose مثالیں۔

مندرجہ ذیل اقسام میں سے **ایک** کا انتخاب کریں اور اسے اپنی ڈائرکٹری میں `compose.yaml` کے بطور محفوظ کریں۔ کسی ریپوزٹری کلون کی ضرورت نہیں ہے۔ اس کے ساتھ ساتھ ایک `.env` فائل بنائیں اس ویرینٹ کے لیے دکھائی گئی قدروں کے ساتھ۔ Compose واضح طور پر ان اقدار کو کنٹینر میں منتقل کرتا ہے۔

`NUXT_SESSION_PASSWORD` کے لیے کم از کم 32 حروف کی بے ترتیب قدر پیدا کریں:

```sh
openssl rand -base64 32
```

اس کلید کو مستحکم رکھیں اور `.env` کی حفاظت کریں۔ اسے ورژن کنٹرول کرنے کا ارتکاب نہ کریں۔ شائع شدہ Crateview تصاویر Linux amd64 کو سپورٹ کرتی ہیں۔

### موجودہ رجسٹری کے ساتھ UI

کنٹینر سے قابل رسائی رجسٹری HTTP API V2 اینڈ پوائنٹ کی ضرورت ہے۔ مثال `.env` (پتہ بدل دیں؛ یو آر ایل میں اسناد شامل نہ کریں):

```dotenv
NUXT_REGISTRY_UPSTREAM=https://registry.example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

```yaml
services:
  frontend:
    image: crateview-registry-ui:next
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

کھولیں **http://localhost:3000**. `127.0.0.1` کے اپ اسٹریم سے مراد کنٹینر ہی ہے۔ مشترکہ Docker نیٹ ورک پر قابل رسائی DNS نام یا سروس ایڈریس استعمال کریں۔

### UI اس کی اپنی رجسٹری کے ساتھ

بغیر تصدیق کے مقامی اسٹیک۔ لوپ بیک پر صرف UI پورٹ شائع ہوتا ہے۔ رجسٹری Docker نیٹ ورک پر رہتی ہے۔ `.env`:

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
    image: crateview-registry-ui:next
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

کھولیں **http://localhost:3000**. رجسٹری ڈیٹا `registry-data` والیوم میں برقرار ہے۔ Push اور pull ایک ہی اصل کے ذریعے ایک تصویر:

```sh
docker pull alpine:latest
docker tag alpine:latest localhost:3000/alpine:latest
docker push localhost:3000/alpine:latest
docker pull localhost:3000/alpine:latest
```

### HTTPS کے ساتھ UI، رجسٹری اور Traefik

اسٹینڈ اسٹیک بشمول Traefik اور Let's Encrypt۔ عوامی طور پر قابل رسائی سرور، دستیاب اور اجازت یافتہ TCP پورٹ 80/443، اور ایک ڈومین کی ضرورت ہے جس کے DNS ریکارڈ اس سرور کی طرف اشارہ کرتے ہیں۔ پورٹ 80 کو HTTP-01 چیلنج کے لیے قابل رسائی رہنا چاہیے۔ `.env`:

```dotenv
CRATEVIEW_DOMAIN=registry.example.com
ACME_EMAIL=admin@example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

اسکیم یا راستے کے بغیر ڈومین داخل کریں۔ تمام مثالی اقدار کو تبدیل کریں۔ پھر انٹرایکٹو طور پر ایک bcrypt `htpasswd` فائل بنائیں۔ `your-user` کو اپنے صارف نام سے تبدیل کریں:

```sh
mkdir -p auth
chmod 700 auth
docker run --rm -it --mount type=bind,src="$(pwd)/auth",dst=/auth \
  --entrypoint htpasswd httpd:2 -Bc /auth/htpasswd your-user
```

پاس ورڈ کا اشارہ کیا گیا ہے اور کمانڈ میں ظاہر نہیں ہوتا ہے۔ `-c` ایک نئی فائل بناتا ہے۔ صارفین کو شامل کرتے وقت اسے چھوڑ دیں۔ `auth` ڈائریکٹری کو محفوظ اور بیک اپ کریں۔ اس `compose.yaml` کو اس کے ساتھ محفوظ کریں:

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
    image: crateview-registry-ui:next
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

اپنے ڈومین کا استعمال کرتے ہوئے **https://registry.example.com** کھولیں اور اپنے بنائے ہوئے رجسٹری صارف کے ساتھ سائن ان کریں۔ Traefik خود بخود سرٹیفکیٹ حاصل کرتا ہے اور HTTP کو HTTPS پر ری ڈائریکٹ کرتا ہے۔ سرٹیفکیٹس `letsencrypt` میں برقرار ہیں، `registry-data` میں رجسٹری ڈیٹا۔ دونوں والیوم کا بیک اپ لیں اور معمول کے شٹ ڈاؤن کے لیے `docker compose down -v` استعمال نہ کریں۔

تمام راستے، بشمول `/v2/`، Crateview سے گزرتے ہیں۔ نہ تو رجسٹری اور نہ ہی UI اپنے میزبان پورٹس شائع کرتا ہے۔ رجسٹری Basic Auth کو نافذ کرتی ہے۔ کسی اضافی Traefik Basic Auth کی ضرورت نہیں ہے۔ Traefik عوامی میزبان کو محفوظ رکھتا ہے اور HTTPS پروٹوکول کو Crateview کو فارورڈ کرتا ہے۔ Docker ساکٹ Traefik کو Docker API تک رسائی فراہم کرتا ہے۔ اس اسٹیک کو ایک قابل اعتماد میزبان پر چلائیں۔

CLI لاگ ان اور تصویر کی منتقلی (ڈومین کو تبدیل کریں):

```sh
docker login registry.example.com --username your-user
docker pull alpine:latest
docker tag alpine:latest registry.example.com/alpine:latest
docker push registry.example.com/alpine:latest
docker pull registry.example.com/alpine:latest
```

CLI UI لاگ ان سے الگ پاس ورڈ کا اشارہ کرتا ہے۔ تفصیلات کے لیے [authentication](/ur/docs/next/authentication) اور [Traefik Docker اور Let’s Encrypt](https://doc.traefik.io/traefik/setup/docker/#tls-certificate-management-lets-encrypt) کے ساتھ دیکھیں۔

## موجودہ رجسٹری کا استعمال کریں۔

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 3844d7605dac80d0d58999201c43e94c644d72c0
npm ci
cp .env.example .env
```

`.env` میں اپ اسٹریم سیٹ کریں، پھر ڈیولپمنٹ موڈ شروع کریں:

```dotenv
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000
```

```sh
npm run dev
```

UI گرم ماڈیول کی تبدیلی کے ساتھ پورٹ 3000 پر چلتا ہے۔ جب فرنٹ اینڈ ایک کنٹینر کے اندر چلتا ہے، `127.0.0.1` کا مطلب ہے وہ کنٹینر۔ اس کے بجائے ایک قابل رسائی سروس ایڈریس، جیسے `http://registry:5000`، استعمال کریں۔

## پروڈکشن نوڈ سرور

```sh
npm run build
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000 HOST=127.0.0.1 PORT=3000 npm start
```

پروڈکشن نوڈ کا عمل اصل ماحولیاتی متغیرات کو پڑھتا ہے، خود بخود `.env` نہیں۔ اس کے مطابق اپنے پروسیس مینیجر یا کنٹینر ماحول کو ترتیب دیں۔ ریموٹ تصدیق شدہ استعمال کے لیے HTTPS پیش کریں۔

Crateview کو تصدیق اور API پراکسینگ کے لیے اس کے Nitro سرور کی ضرورت ہے۔ ایک مستحکم ویب سائٹ کے طور پر صرف فرنٹ اینڈ فائلوں کو پیش کرنا ایک ورکنگ رجسٹری UI فراہم نہیں کرے گا۔

سروس کو اپنی مشین سے آگے ظاہر کرنے سے پہلے [configuration](/ur/docs/next/configuration) کے ساتھ جاری رکھیں۔

## ماخذ سے بنائیں

مقامی تعمیر کے لیے، سورس ریپوزٹری سے Compose فائلیں استعمال کریں:

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 3844d7605dac80d0d58999201c43e94c644d72c0
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```
