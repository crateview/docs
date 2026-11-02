---
title: "安装"
description: "启动完整的堆栈或将 UI 连接到现有镜像仓库。"
---

示例应匹配此版本源码；安装归档版本时不要克隆默认分支。

预览容器需先从固定源码修订本地构建，不假定存在已发布预览镜像。

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 3844d7605dac80d0d58999201c43e94c644d72c0
docker build -t crateview-registry-ui:next .
```


## 要求

使用 Node.js 26.13.0 或 v26 中的更高版本进行本地开发，或使用 Docker 和 Compose 进行完整堆栈。现有镜像仓库必须公开镜像仓库 HTTP API V2；捆绑的堆栈使用官方 `registry:3.1.2` 映像。

## Docker Compose 示例

选择以下变体之一并将其另存为 `compose.yaml` 在其自己的目录中。不需要存储库克隆。在其旁边创建一个 `.env` 文件，并包含该变体显示的值。 Compose 显式地将这些值传递给容器。

为 `NUXT_SESSION_PASSWORD` 生成至少 32 个字符的随机值：

```sh
openssl rand -base64 32
```

保持该密钥稳定并保护`.env`；不要将其提交给版本控制。已发布的 Crateview 映像支持 Linux amd64。

### 具有现有镜像仓库的 UI

需要可从容器访问的镜像仓库 HTTP API V2 端点。示例 `.env`（替换地址；不在 URL 中包含凭据）：

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

Open **http://localhost:3000**. `127.0.0.1` 的上游指的是容器本身。使用共享 Docker 网络上可访问的 DNS 名称或服务地址。

### 具有自己的镜像仓库的 UI

无需身份验证的本地堆栈。仅 UI 端口在环回上发布；镜像仓库保留在 Docker 网络上。 `.env`：

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

打开 **http://localhost:3000**. 镜像仓库数据保留在 `registry-data` 卷中。 Push 和 pull 的图像经过相同的原点：

```sh
docker pull alpine:latest
docker tag alpine:latest localhost:3000/alpine:latest
docker push localhost:3000/alpine:latest
docker pull localhost:3000/alpine:latest
```

### 使用 HTTPS 的 UI、镜像仓库和 Traefik

独立堆栈包括 Traefik 和 Let's Encrypt。需要可公开访问的服务器、可用且允许的 TCP 端口 80/443 以及 DNS 记录指向该服务器的域。对于 HTTP-01 质询，端口 80 必须保持可达。 `.env`：

```dotenv
CRATEVIEW_DOMAIN=registry.example.com
ACME_EMAIL=admin@example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

输入不带方案或路径的域。替换所有示例值。然后交互创建bcrypt `htpasswd`文件；将 `your-user` 替换为您的用户名：

```sh
mkdir -p auth
chmod 700 auth
docker run --rm -it --mount type=bind,src="$(pwd)/auth",dst=/auth \
  --entrypoint htpasswd httpd:2 -Bc /auth/htpasswd your-user
```

系统会提示输入密码，但不会出现在命令中。 `-c` 创建一个新文件；添加用户时省略它。保护并备份`auth`目录。将此 `compose.yaml` 保存在旁边：

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

使用您的域打开 **https://registry.example.com** 并使用您创建的镜像仓库用户登录。 Traefik自动获取证书并将HTTP重定向到HTTPS。证书保留在 `letsencrypt` 中，镜像仓库数据保留在 `registry-data` 中。备份两个卷，并且不要使用 `docker compose down -v` 进行例行关闭。

所有路径（包括 `/v2/`）均经过 Crateview。镜像仓库和 UI 都不会发布自己的主机端口。镜像仓库强制执行Basic Auth；不需要额外的 Traefik Basic Auth。 Traefik保留公共主机，并将HTTPS协议转发给Crateview。 Docker 插座使 Traefik 能够访问 Docker API；在受信任的主机上运行此堆栈。

CLI登录和图像传输（替换域）：

```sh
docker login registry.example.com --username your-user
docker pull alpine:latest
docker tag alpine:latest registry.example.com/alpine:latest
docker push registry.example.com/alpine:latest
docker pull registry.example.com/alpine:latest
```

CLI 与 UI 登录分开提示输入密码。有关详细信息，请参阅[authentication](/zh-CN/docs/next/authentication) 和 [Traefik with Docker 和 Let’s Encrypt](https://doc.traefik.io/traefik/setup/docker/#tls-certificate-management-lets-encrypt)]。

## 使用现有镜像仓库

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 3844d7605dac80d0d58999201c43e94c644d72c0
npm ci
cp .env.example .env
```

设置上游为`.env`，然后启动开发模式：

```dotenv
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000
```

```sh
npm run dev
```

UI 在端口 3000 上运行，支持热模块更换。当前端在容器内运行时，`127.0.0.1` 表示该容器。请改用可访问的服务地址，例如 `http://registry:5000`。

## 生产节点服务器

```sh
npm run build
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000 HOST=127.0.0.1 PORT=3000 npm start
```

生产节点进程读取实际环境变量，而不是自动读取 `.env`。相应地配置您的流程管理器或容器环境。为远程身份验证使用提供 HTTPS。

Crateview 需要其 Nitro 服务器进行身份验证和 API 代理。仅将前端文件作为静态网站提供服务将不会提供有效的镜像仓库 UI。

在将服务公开到您自己的计算机之外之前，请继续 [configuration](/zh-CN/docs/next/configuration)]。

## 从源代码构建

对于本地构建，请使用源存储库中的 Compose 文件：

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 3844d7605dac80d0d58999201c43e94c644d72c0
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```
