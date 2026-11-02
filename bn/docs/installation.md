---
title: "ইনস্টলেশন"
description: "সম্পূর্ণ স্ট্যাক শুরু করুন বা একটি বিদ্যমান রেজিস্ট্রিতে UI সংযোগ করুন।"
---

## প্রয়োজনীয়তা

স্থানীয় উন্নয়নের জন্য Node.js 24.22.2 বা v24 এর মধ্যে নতুন ব্যবহার করুন, অথবা সম্পূর্ণ স্ট্যাকের জন্য Compose সহ Docker ব্যবহার করুন। একটি বিদ্যমান রেজিস্ট্রি অবশ্যই রেজিস্ট্রি HTTP API V2 প্রকাশ করবে; বান্ডিল স্ট্যাক অফিসিয়াল `registry:3.1.2` ইমেজ ব্যবহার করে।

## Docker Compose উদাহরণ

নিম্নলিখিত ভেরিয়েন্টগুলির মধ্যে **একটি** চয়ন করুন এবং এটিকে তার নিজস্ব ডিরেক্টরিতে `compose.yaml` হিসাবে সংরক্ষণ করুন৷ কোন সংগ্রহস্থল ক্লোন প্রয়োজন নেই. সেই ভেরিয়েন্টের জন্য দেখানো মানগুলির সাথে এটির পাশাপাশি একটি `.env` ফাইল তৈরি করুন৷ Compose স্পষ্টভাবে এই মানগুলি কন্টেইনারে পাস করে।

`NUXT_SESSION_PASSWORD` এর জন্য কমপক্ষে 32টি অক্ষরের একটি এলোমেলো মান তৈরি করুন:

```sh
openssl rand -base64 32
```

এই কী স্থিতিশীল রাখুন এবং `.env` রক্ষা করুন; সংস্করণ নিয়ন্ত্রণে এটি প্রতিশ্রুতিবদ্ধ করবেন না। প্রকাশিত Crateview চিত্রগুলি Linux amd64 সমর্থন করে।

### একটি বিদ্যমান রেজিস্ট্রি সহ UI

একটি রেজিস্ট্রি HTTP API V2 কন্টেইনার থেকে পৌঁছানো যায় এমন এন্ডপয়েন্ট প্রয়োজন৷ উদাহরণ `.env` (ঠিকানা প্রতিস্থাপন করুন; URL-এ শংসাপত্র অন্তর্ভুক্ত করবেন না):

```dotenv
NUXT_REGISTRY_UPSTREAM=https://registry.example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

```yaml
services:
  frontend:
    image: ghcr.io/crateview/registry-ui:v0.1.0
    ports:
      - "127.0.0.1:3000:3000"
    environment:
      NUXT_REGISTRY_UPSTREAM: ${NUXT_REGISTRY_UPSTREAM:?Set NUXT_REGISTRY_UPSTREAM}
      NUXT_SESSION_PASSWORD: ${NUXT_SESSION_PASSWORD:?Set NUXT_SESSION_PASSWORD}
    restart: unless-stopped
```

```sh
docker compose config --quiet
docker compose pull
docker compose up -d
```

খুলুন **http://localhost:3000**. `127.0.0.1` এর একটি আপস্ট্রিম ধারককেই বোঝায়। একটি শেয়ার করা Docker নেটওয়ার্কে একটি পৌঁছানোযোগ্য DNS নাম বা একটি পরিষেবা ঠিকানা ব্যবহার করুন৷

### নিজস্ব রেজিস্ট্রি সহ UI

প্রমাণীকরণ ছাড়া স্থানীয় স্ট্যাক. লুপব্যাকে শুধুমাত্র UI পোর্ট প্রকাশিত হয়; রেজিস্ট্রি Docker নেটওয়ার্কে থাকে। `.env`:

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
    image: ghcr.io/crateview/registry-ui:v0.1.0
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
docker compose pull
docker compose up -d
```

খুলুন **http://localhost:3000**. রেজিস্ট্রি ডেটা `registry-data` ভলিউমে টিকে থাকে। Push এবং pull একই উত্সের মাধ্যমে একটি চিত্র:

```sh
docker pull alpine:latest
docker tag alpine:latest localhost:3000/alpine:latest
docker push localhost:3000/alpine:latest
docker pull localhost:3000/alpine:latest
```

### HTTPS সহ UI, রেজিস্ট্রি এবং Traefik

Traefik এবং Let's Encrypt সহ স্বতন্ত্র স্ট্যাক। একটি সর্বজনীনভাবে পৌঁছানোর যোগ্য সার্ভার প্রয়োজন, উপলব্ধ এবং অনুমোদিত TCP পোর্ট 80/443, এবং একটি ডোমেন যার DNS রেকর্ডগুলি সেই সার্ভারের দিকে নির্দেশ করে৷ পোর্ট 80 অবশ্যই HTTP-01 চ্যালেঞ্জের জন্য পৌঁছানো যাবে। `.env`:

```dotenv
CRATEVIEW_DOMAIN=registry.example.com
ACME_EMAIL=admin@example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

কোনো স্কিম বা পাথ ছাড়াই ডোমেনে প্রবেশ করুন। সমস্ত উদাহরণ মান প্রতিস্থাপন করুন। তারপর ইন্টারেক্টিভভাবে একটি bcrypt `htpasswd` ফাইল তৈরি করুন; আপনার ব্যবহারকারীর নাম দিয়ে `your-user` প্রতিস্থাপন করুন:

```sh
mkdir -p auth
chmod 700 auth
docker run --rm -it --mount type=bind,src="$(pwd)/auth",dst=/auth \
  --entrypoint htpasswd httpd:2 -Bc /auth/htpasswd your-user
```

পাসওয়ার্ডের জন্য অনুরোধ করা হয় এবং কমান্ডে প্রদর্শিত হয় না। `-c` একটি নতুন ফাইল তৈরি করে; ব্যবহারকারীদের যোগ করার সময় এটি বাদ দিন। `auth` ডিরেক্টরি রক্ষা করুন এবং ব্যাক আপ করুন। এটির পাশে এই `compose.yaml` সংরক্ষণ করুন:

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
    image: ghcr.io/crateview/registry-ui:v0.1.0
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
docker compose pull
docker compose up -d
```

আপনার ডোমেন ব্যবহার করে **https://registry.example.com** খুলুন এবং আপনার তৈরি করা রেজিস্ট্রি ব্যবহারকারীর সাথে সাইন ইন করুন। Traefik স্বয়ংক্রিয়ভাবে শংসাপত্র পায় এবং HTTP-কে HTTPS-এ পুনঃনির্দেশ করে। শংসাপত্র `letsencrypt`, `registry-data`-এ রেজিস্ট্রি ডেটা বজায় থাকে। উভয় ভলিউম ব্যাক আপ করুন এবং রুটিন শাটডাউনের জন্য `docker compose down -v` ব্যবহার করবেন না।

`/v2/` সহ সমস্ত পথ, Crateview এর মধ্য দিয়ে যায়। রেজিস্ট্রি বা UI কেউই নিজস্ব হোস্ট পোর্ট প্রকাশ করে না। রেজিস্ট্রি Basic Auth প্রয়োগ করে; কোন অতিরিক্ত Traefik Basic Auth প্রয়োজন নেই৷ Traefik পাবলিক হোস্ট সংরক্ষণ করে এবং HTTPS প্রোটোকলকে Crateview-এ ফরোয়ার্ড করে। Docker সকেট Traefik কে Docker API অ্যাক্সেস দেয়; একটি বিশ্বস্ত হোস্ট এই স্ট্যাক চালান.

CLI লগইন এবং ছবি স্থানান্তর (ডোমেন প্রতিস্থাপন):

```sh
docker login registry.example.com --username your-user
docker pull alpine:latest
docker tag alpine:latest registry.example.com/alpine:latest
docker push registry.example.com/alpine:latest
docker pull registry.example.com/alpine:latest
```

CLI UI লগইন থেকে আলাদাভাবে পাসওয়ার্ডের জন্য অনুরোধ করে। বিস্তারিত জানার জন্য [প্রমাণকরণ](/bn/docs/authentication) এবং [Traefik Docker এবং Let’s Encrypt](https://doc.traefik.io/traefik/setup/docker/#tls-certificate-management-lets-encrypt) সহ দেখুন।

## একটি বিদ্যমান রেজিস্ট্রি ব্যবহার করুন

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
npm ci
cp .env.example .env
```

`.env`-এ আপস্ট্রিম সেট করুন, তারপর ডেভেলপমেন্ট মোড শুরু করুন:

```dotenv
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000
```

```sh
npm run dev
```

UI হট মডিউল প্রতিস্থাপন সহ পোর্ট 3000 এ চলে। যখন ফ্রন্টএন্ড একটি পাত্রের ভিতরে চলে, তখন `127.0.0.1` মানে সেই ধারক। পরিবর্তে একটি পৌঁছানো যায় এমন পরিষেবার ঠিকানা ব্যবহার করুন, যেমন `http://registry:5000`,

## উত্পাদন নোড সার্ভার

```sh
npm run build
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000 HOST=127.0.0.1 PORT=3000 npm start
```

উৎপাদন নোড প্রক্রিয়া প্রকৃত পরিবেশের ভেরিয়েবল পড়ে, স্বয়ংক্রিয়ভাবে `.env` নয়। সেই অনুযায়ী আপনার প্রসেস ম্যানেজার বা ধারক পরিবেশ কনফিগার করুন। দূরবর্তী প্রমাণীকৃত ব্যবহারের জন্য HTTPS পরিবেশন করুন।

Crateview প্রমাণীকরণ এবং API প্রক্সি করার জন্য এর Nitro সার্ভার প্রয়োজন৷ একটি স্ট্যাটিক ওয়েবসাইট হিসাবে শুধুমাত্র ফ্রন্টএন্ড ফাইল পরিবেশন করা একটি কার্যকরী রেজিস্ট্রি UI প্রদান করবে না।

আপনার নিজের মেশিনের বাইরে পরিষেবাটি প্রকাশ করার আগে [configuration](/bn/docs/configuration) এর সাথে চালিয়ে যান।

## উৎস থেকে তৈরি করুন

একটি স্থানীয় বিল্ডের জন্য, উত্স সংগ্রহস্থল থেকে Compose ফাইলগুলি ব্যবহার করুন:

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```
