---
title: "インストール"
description: "完全なスタックを開始するか、UI を既存のレジストリに接続します。"
---

## 要件

ローカル開発には v24 内の Node.js 24.22.2 以降を使用するか、完全なスタックには Compose で Docker を使用します。既存のレジストリは、レジストリ HTTP API V2 を公開する必要があります。バンドルされたスタックは、公式の `registry:3.1.2` イメージを使用します。

## Docker Compose の例

次のバリアントから **1 つ**を選択し、それを `compose.yaml` として独自のディレクトリに保存します。リポジトリのクローンは必要ありません。そのバリアントに示されている値を含む `.env` ファイルを一緒に作成します。 Compose はこれらの値をコンテナに明示的に渡します。

`NUXT_SESSION_PASSWORD` に対して少なくとも 32 文字のランダムな値を生成します。

```sh
openssl rand -base64 32
```

このキーを安定した状態に保ち、`.env` を保護します。バージョン管理にコミットしないでください。公開された Crateview イメージは Linux amd64 をサポートします。

### 既存のレジストリを使用した UI

コンテナから到達可能なレジストリ HTTP API V2 エンドポイントが必要です。例 `.env` (アドレスを置き換えます。URL には資格情報を含めないでください):

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

オープン **http://localhost:3000**. `127.0.0.1` の上流はコンテナ自体を指します。共有 Docker ネットワーク上の到達可能な DNS 名またはサービス アドレスを使用します。

### 独自のレジストリを備えた UI

認証なしのローカルスタック。 UI ポートのみがループバックで公開されます。レジストリは Docker ネットワーク上に残ります。 `.env`:

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

**http://localhost:3000**. を開く レジストリ データは `registry-data` ボリュームに保持されます。 Push と pull は同じオリジンによる画像:

```sh
docker pull alpine:latest
docker tag alpine:latest localhost:3000/alpine:latest
docker push localhost:3000/alpine:latest
docker pull localhost:3000/alpine:latest
```

### HTTPS を使用した UI、レジストリ、Traefik

Traefik と Let's Encrypt を含むスタンドアロン スタック。パブリックにアクセス可能なサーバー、使用可能かつ許可された TCP ポート 80/443、およびそのサーバーを指す DNS レコードを持つドメインが必要です。ポート 80 は、HTTP-01 チャレンジに対して到達可能な状態を維持する必要があります。 `.env`:

```dotenv
CRATEVIEW_DOMAIN=registry.example.com
ACME_EMAIL=admin@example.com
NUXT_SESSION_PASSWORD=REPLACE_WITH_GENERATED_SECRET
```

スキームやパスを指定せずにドメインを入力します。すべての例の値を置き換えます。次に、bcrypt `htpasswd` ファイルを対話的に作成します。 `your-user` を自分のユーザー名に置き換えます。

```sh
mkdir -p auth
chmod 700 auth
docker run --rm -it --mount type=bind,src="$(pwd)/auth",dst=/auth \
  --entrypoint htpasswd httpd:2 -Bc /auth/htpasswd your-user
```

パスワードの入力を求められますが、コマンドには表示されません。 `-c` は新しいファイルを作成します。ユーザーを追加する場合は省略します。 `auth` ディレクトリを保護してバックアップします。この `compose.yaml` を一緒に保存します。

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

ドメインを使用して **https://registry.example.com** を開き、作成したレジストリ ユーザーでサインインします。 Traefik は証明書を自動的に取得し、HTTP を HTTPS にリダイレクトします。証明書は `letsencrypt` に保持され、レジストリ データは `registry-data` に保持されます。両方のボリュームをバックアップし、定期的なシャットダウンには `docker compose down -v` を使用しないでください。

`/v2/` を含むすべてのパスは Crateview を通過します。レジストリも UI も独自のホスト ポートを公開しません。レジストリは Basic Auth を適用します。追加の Traefik Basic Auth は必要ありません。 Traefik はパブリック ホストを保持し、HTTPS プロトコルを Crateview に転送します。 Docker ソケットにより、Traefik は Docker API にアクセスできるようになります。このスタックを信頼できるホスト上で実行します。

CLI ログインとイメージ転送 (ドメインを置き換えます):

```sh
docker login registry.example.com --username your-user
docker pull alpine:latest
docker tag alpine:latest registry.example.com/alpine:latest
docker push registry.example.com/alpine:latest
docker pull registry.example.com/alpine:latest
```

CLI は、UI ログインとは別にパスワードの入力を求めます。詳細については、[authentication](/ja/docs/authentication) および [Traefik with Docker および Let’s Encrypt](https://doc.traefik.io/traefik/setup/docker/#tls-certificate-management-lets-encrypt)] を参照してください。

## 既存のレジストリを使用する

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
npm ci
cp .env.example .env
```

`.env` でアップストリームを設定し、開発モードを開始します。

```dotenv
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000
```

```sh
npm run dev
```

UI は、ホット モジュール交換によりポート 3000 で実行されます。フロントエンドがコンテナ内で実行される場合、`127.0.0.1` はそのコンテナを意味します。代わりに、`http://registry:5000` などの到達可能なサービス アドレスを使用してください。

## 本番ノードサーバー

```sh
npm run build
NUXT_REGISTRY_UPSTREAM=http://127.0.0.1:5000 HOST=127.0.0.1 PORT=3000 npm start
```

運用ノード プロセスは、`.env` を自動的に読み取るのではなく、実際の環境変数を読み取ります。それに応じてプロセス マネージャーまたはコンテナー環境を構成します。リモート認証された使用のために HTTPS を提供します。

Crateview は、認証と API プロキシのために Nitro サーバーを必要とします。フロントエンド ファイルのみを静的 Web サイトとして提供すると、機能するレジストリ UI は提供されません。

自分のマシン以外にサービスを公開する前に、[configuration](/ja/docs/configuration)] に進みます。

## ソースからビルドする

ローカル ビルドの場合は、ソース リポジトリから Compose ファイルを使用します。

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```
