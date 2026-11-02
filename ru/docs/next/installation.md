---
title: "Установка"
description: "Запустите полный стек или подключите пользовательский интерфейс к существующему реестру."
---

Используйте ревизию этой версии; не клонируйте основную ветку для архивной установки.

Предварительные контейнеры сначала собираются локально из закреплённой ревизии; опубликованный образ не предполагается.

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout bef44098e9acfe2aaa39c0ba0bbaa768b7853241
docker build -t crateview-registry-ui:next .
```


## Требования

Используйте Node.js 26.13.0 или новее в версии 26 для локальной разработки или Docker с Compose для полного стека. Существующий реестр должен предоставлять реестр HTTP API V2; в комплекте стека используется официальный образ `registry:3.1.2`.

## Docker Compose примеры

Выберите **один** из следующих вариантов и сохраните его как `compose.yaml` в отдельном каталоге. Клон репозитория не требуется. Создайте рядом с ним файл `.env` со значениями, указанными для этого варианта. Compose явно передает эти значения в контейнер.

Создайте случайное значение длиной не менее 32 символов для `NUXT_SESSION_PASSWORD`:

```sh
openssl rand -base64 32
```

Сохраняйте стабильность этого ключа и защитите `.env`; не фиксируйте его в системе контроля версий. Опубликованные образы Crateview поддерживают Linux amd64.

### Пользовательский интерфейс с существующим реестром

Требуется конечная точка реестра HTTP API V2, доступная из контейнера. Пример `.env` (замените адрес; не включайте учетные данные в URL-адрес):

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

Открыть **http://localhost:3000**. Восходящий элемент `127.0.0.1` относится к самому контейнеру. Используйте доступное DNS-имя или адрес службы в общей сети Docker.

### Пользовательский интерфейс со своим реестром

Локальный стек без аутентификации. В режиме обратной связи публикуется только порт пользовательского интерфейса; реестр остается в сети Docker. `.env`:

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

Открыть **http://localhost:3000**. Данные реестра сохраняются в томе `registry-data`. Push и pull — изображения из того же источника:

```sh
docker pull alpine:latest
docker tag alpine:latest localhost:3000/alpine:latest
docker push localhost:3000/alpine:latest
docker pull localhost:3000/alpine:latest
```

### Пользовательский интерфейс, реестр и Traefik с HTTPS

Автономный стек, включающий Traefik и Let's Encrypt. Требуется общедоступный сервер, доступные и разрешенные TCP-порты 80/443 и домен, записи DNS которого указывают на этот сервер. Порт 80 должен оставаться доступным для запроса HTTP-01. `.env`:

```dotenv
CRATEVIEW_DOMAIN=registry.example.com
ACME_EMAIL=admin@example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

Введите домен без схемы или пути. Замените все значения примера. Затем в интерактивном режиме создайте файл bcrypt `htpasswd`; замените `your-user` на свое имя пользователя:

```sh
mkdir -p auth
chmod 700 auth
docker run --rm -it --mount type=bind,src="$(pwd)/auth",dst=/auth \
  --entrypoint htpasswd httpd:2 -Bc /auth/htpasswd your-user
```

Пароль запрашивается, но не отображается в команде. `-c` создает новый файл; опустите его при добавлении пользователей. Защитите и создайте резервную копию каталога `auth`. Сохраните этот `compose.yaml` рядом с ним:

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

Откройте **https://registry.example.com**, используя свой домен, и войдите в систему, используя созданного вами пользователя реестра. Traefik автоматически получает сертификат и перенаправляет HTTP на HTTPS. Сертификаты сохраняются в `letsencrypt`, данные реестра — в `registry-data`. Создайте резервную копию обоих томов и не используйте `docker compose down -v` для планового завершения работы.

Все пути, включая `/v2/`, проходят через Crateview. Ни реестр, ни пользовательский интерфейс не публикуют собственные порты хоста. Реестр применяет Basic Auth; никаких дополнительных Traefik Basic Auth не требуется. Traefik сохраняет общедоступный хост и перенаправляет протокол HTTPS на Crateview. Сокет Docker предоставляет Traefik доступ к Docker API; запустите этот стек на доверенном хосте.

Вход в CLI и перенос образа (замена домена):

```sh
docker login registry.example.com --username your-user
docker pull alpine:latest
docker tag alpine:latest registry.example.com/alpine:latest
docker push registry.example.com/alpine:latest
docker pull registry.example.com/alpine:latest
```

Интерфейс командной строки запрашивает пароль отдельно от входа в пользовательский интерфейс. Подробности см. в [authentication](/ru/docs/next/authentication) и [Traefik с Docker и Let’s Encrypt](https://doc.traefik.io/traefik/setup/docker/#tls-certificate-management-lets-encrypt)].

## Использовать существующий реестр

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout bef44098e9acfe2aaa39c0ba0bbaa768b7853241
npm ci
cp .env.example .env
```

Установите восходящий поток в `.env`, затем запустите режим разработки:

```dotenv
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000
```

```sh
npm run dev
```

Пользовательский интерфейс работает на порту 3000 с горячей заменой модуля. Когда интерфейс работает внутри контейнера, `127.0.0.1` означает этот контейнер. Вместо этого используйте доступный адрес службы, например `http://registry:5000`.

## Сервер производственного узла

```sh
npm run build
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000 HOST=127.0.0.1 PORT=3000 npm start
```

Процесс производственного узла считывает фактические переменные среды, а не `.env` автоматически. Настройте свой менеджер процессов или контейнерную среду соответствующим образом. Обслуживание HTTPS для удаленного использования с аутентификацией.

Crateview нужен сервер Nitro для аутентификации и проксирования API. Обслуживание только файлов внешнего интерфейса в качестве статического веб-сайта не обеспечит работающий пользовательский интерфейс реестра.

Продолжайте с [configuration](/ru/docs/next/configuration), прежде чем предоставлять доступ к службе за пределами вашего собственного компьютера.

## Сборка из исходного кода

Для локальной сборки используйте файлы Compose из исходного репозитория:

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout bef44098e9acfe2aaa39c0ba0bbaa768b7853241
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```
