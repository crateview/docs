---
title: "इंस्टालेशन"
description: "संपूर्ण स्टैक प्रारंभ करें या UI को किसी मौजूदा रजिस्ट्री से कनेक्ट करें।"
---

इसी संस्करण का स्रोत लें; संग्रह स्थापना के लिए डिफ़ॉल्ट शाखा क्लोन न करें।

<!-- release:installation:start -->
पूर्वावलोकन कंटेनर पहले निश्चित स्रोत से स्थानीय बनाएँ; प्रकाशित इमेज न मानें।

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 9954f9c33e34c65bab19378a9b488e9b8ed13b6d
docker build -t crateview-registry-ui:next .
```
<!-- release:installation:end -->


## आवश्यकताएं

स्थानीय विकास के लिए Node.js 26.13.0 या v26 के भीतर नए का उपयोग करें, या संपूर्ण स्टैक के लिए Compose के साथ Docker का उपयोग करें। मौजूदा रजिस्ट्री को रजिस्ट्री HTTP API V2 को उजागर करना होगा; बंडल किया गया स्टैक आधिकारिक `registry:3.1.2` छवि का उपयोग करता है।

## Docker Compose उदाहरण

निम्नलिखित में से **एक** चुनें और इसे अपनी निर्देशिका में `compose.yaml` के रूप में सहेजें। किसी रिपोजिटरी क्लोन की आवश्यकता नहीं है. उस वैरिएंट के लिए दिखाए गए मानों के साथ एक `.env` फ़ाइल बनाएं। Compose स्पष्ट रूप से इन मानों को कंटेनर में भेजता है।

`NUXT_SESSION_PASSWORD` के लिए कम से कम 32 वर्णों का यादृच्छिक मान उत्पन्न करें:

```sh
openssl rand -base64 32
```

इस कुंजी को स्थिर रखें और `.env` की सुरक्षा करें; इसे संस्करण नियंत्रण के लिए प्रतिबद्ध न करें. प्रकाशित Crateview छवियाँ Linux amd64 का समर्थन करती हैं।

### मौजूदा रजिस्ट्री के साथ यूआई

कंटेनर से पहुंच योग्य रजिस्ट्री HTTP API V2 समापन बिंदु की आवश्यकता है। उदाहरण `.env` (पता बदलें; URL में क्रेडेंशियल शामिल न करें):

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

खुला **http://localhost:3000**. `127.0.0.1` का एक अपस्ट्रीम कंटेनर को ही संदर्भित करता है। साझा Docker नेटवर्क पर पहुंच योग्य DNS नाम या सेवा पते का उपयोग करें।

### अपनी स्वयं की रजिस्ट्री के साथ यूआई

प्रमाणीकरण के बिना स्थानीय स्टैक. केवल यूआई पोर्ट लूपबैक पर प्रकाशित होता है; रजिस्ट्री Docker नेटवर्क पर रहती है। `.env`:

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

खोलें **http://localhost:3000**. रजिस्ट्री डेटा `registry-data` वॉल्यूम में बना रहता है। Push और pull एक ही मूल से एक छवि:

```sh
docker pull alpine:latest
docker tag alpine:latest localhost:3000/alpine:latest
docker push localhost:3000/alpine:latest
docker pull localhost:3000/alpine:latest
```

### HTTPS के साथ यूआई, रजिस्ट्री और Traefik

Traefik और लेट्स एनक्रिप्ट सहित स्टैंडअलोन स्टैक। एक सार्वजनिक रूप से पहुंच योग्य सर्वर, उपलब्ध और अनुमत टीसीपी पोर्ट 80/443, और एक डोमेन की आवश्यकता होती है जिसका DNS रिकॉर्ड उस सर्वर पर इंगित करता है। HTTP-01 चुनौती के लिए पोर्ट 80 पहुंच योग्य रहना चाहिए। `.env`:

```dotenv
CRATEVIEW_DOMAIN=registry.example.com
ACME_EMAIL=admin@example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

किसी स्कीम या पथ के बिना डोमेन दर्ज करें. सभी उदाहरण मान बदलें. फिर अंतःक्रियात्मक रूप से एक bcrypt `htpasswd` फ़ाइल बनाएं; `your-user` को अपने उपयोगकर्ता नाम से बदलें:

```sh
mkdir -p auth
chmod 700 auth
docker run --rm -it --mount type=bind,src="$(pwd)/auth",dst=/auth \
  --entrypoint htpasswd httpd:2 -Bc /auth/htpasswd your-user
```

पासवर्ड के लिए संकेत दिया गया है और यह कमांड में दिखाई नहीं देता है। `-c` एक नई फ़ाइल बनाता है; उपयोगकर्ता जोड़ते समय इसे छोड़ दें. `auth` निर्देशिका को सुरक्षित रखें और उसका बैकअप लें। इस `compose.yaml` को इसके साथ सहेजें:

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

अपने डोमेन का उपयोग करके **https://registry.example.com** खोलें और आपके द्वारा बनाए गए रजिस्ट्री उपयोगकर्ता के साथ साइन इन करें। Traefik स्वचालित रूप से प्रमाणपत्र प्राप्त करता है और HTTP को HTTPS पर पुनर्निर्देशित करता है। प्रमाणपत्र `letsencrypt` में बने रहते हैं, रजिस्ट्री डेटा `registry-data` में रहता है। दोनों वॉल्यूम का बैकअप लें और नियमित शटडाउन के लिए `docker compose down -v` का उपयोग न करें।

`/v2/` सहित सभी पथ, Crateview से होकर गुजरते हैं। न तो रजिस्ट्री और न ही यूआई अपने स्वयं के होस्ट पोर्ट प्रकाशित करता है। रजिस्ट्री Basic Auth लागू करती है; किसी अतिरिक्त Traefik Basic Auth की आवश्यकता नहीं है। Traefik सार्वजनिक होस्ट को सुरक्षित रखता है और HTTPS प्रोटोकॉल को Crateview पर अग्रेषित करता है। Docker सॉकेट Traefik को Docker API तक पहुंच प्रदान करता है; इस स्टैक को किसी विश्वसनीय होस्ट पर चलाएँ।

सीएलआई लॉगिन और छवि स्थानांतरण (डोमेन बदलें):

```sh
docker login registry.example.com --username your-user
docker pull alpine:latest
docker tag alpine:latest registry.example.com/alpine:latest
docker push registry.example.com/alpine:latest
docker pull registry.example.com/alpine:latest
```

सीएलआई यूआई लॉगिन से अलग पासवर्ड के लिए संकेत देता है। विवरण के लिए [प्रमाणीकरण](/hi/docs/next/authentication) और [Traefik के साथ Docker और Let’s Encrypt](https://doc.traefik.io/traefik/setup/docker/#tls-certificate-management-lets-encrypt) देखें।

## किसी मौजूदा रजिस्ट्री का उपयोग करें

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 9954f9c33e34c65bab19378a9b488e9b8ed13b6d
npm ci
cp .env.example .env
```

अपस्ट्रीम को `.env` में सेट करें, फिर विकास मोड शुरू करें:

```dotenv
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000
```

```sh
npm run dev
```

यूआई हॉट मॉड्यूल रिप्लेसमेंट के साथ पोर्ट 3000 पर चलता है। जब फ्रंटएंड किसी कंटेनर के अंदर चलता है, तो `127.0.0.1` का मतलब वह कंटेनर होता है। इसके बजाय पहुंच योग्य सेवा पते, जैसे `http://registry:5000`, का उपयोग करें।

## उत्पादन नोड सर्वर

```sh
npm run build
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000 HOST=127.0.0.1 PORT=3000 npm start
```

उत्पादन नोड प्रक्रिया वास्तविक पर्यावरण चर को पढ़ती है, न कि `.env` को स्वचालित रूप से। अपने प्रक्रिया प्रबंधक या कंटेनर वातावरण को तदनुसार कॉन्फ़िगर करें। दूरस्थ प्रमाणित उपयोग के लिए HTTPS परोसें।

Crateview को प्रमाणीकरण और API प्रॉक्सी के लिए अपने Nitro सर्वर की आवश्यकता है। केवल फ़्रंटएंड फ़ाइलों को एक स्थिर वेबसाइट के रूप में प्रस्तुत करने से कार्यशील रजिस्ट्री यूआई उपलब्ध नहीं होगी।

अपनी मशीन से परे सेवा को उजागर करने से पहले [configuration](/hi/docs/next/configuration) के साथ जारी रखें।

## स्रोत से निर्माण करें

स्थानीय निर्माण के लिए, स्रोत भंडार से Compose फ़ाइलों का उपयोग करें:

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 9954f9c33e34c65bab19378a9b488e9b8ed13b6d
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```

<a id="pwa"></a>

## ऐप के रूप में इंस्टॉल करें (PWA)

समर्थित ब्राउज़र के मूल इंस्टॉल या होम स्क्रीन पर जोड़ें विकल्प से Crateview इंस्टॉल करें। यह अलग विंडो में खुलता है, लेकिन Nitro सर्वर और पहुँच योग्य रजिस्ट्री अब भी आवश्यक हैं।

प्रोडक्शन में HTTPS उपयोग करें; स्थानीय परीक्षण के लिए localhost उपयुक्त है। service worker केवल प्रोडक्शन बिल्ड में सक्रिय है, `npm run dev` के दौरान नहीं। परीक्षण से पहले प्रोडक्शन सर्वर बिल्ड करके चलाएँ।

सफल ऑनलाइन विज़िट और service worker इंस्टॉल होने के बाद बिना नेटवर्क ऐप खोलने या रीफ़्रेश करने पर कनेक्शन संदेश दिखता है। नेटवर्क लौटने पर पुनः प्रयास मूल URL लोड करता है। सहेजी गई भाषा ब्राउज़र भाषाओं से पहले चुनी जाती है, और विकल्प अंग्रेज़ी है; अरबी और उर्दू RTL उपयोग करती हैं।

ऑफ़लाइन पेज केवल ऐप के दस्तावेज़ नेविगेशन के लिए है। रजिस्ट्री, खाते और सत्र के उत्तर Cache Storage में सहेजे नहीं जाते; ऑफ़लाइन रजिस्ट्री ब्राउज़िंग उपलब्ध नहीं है। खुले पेजों पर सामान्य त्रुटि व्यवहार बना रहता है। API और `/v2/` ट्रैफ़िक को ऑफ़लाइन HTML से नहीं बदला जाता और worker अपडेट खुले टैब को जबरन रीफ़्रेश नहीं करते।
