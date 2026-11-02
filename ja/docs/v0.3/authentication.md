---
title: "認証"
description: "匿名アクセス、Basic Auth、暗号化セッション、HTTPS。"
---

## サポートされている認証

匿名アクセスと標準 htpasswd Basic Auth に対応します。外部 Bearer サービスと OIDC は未対応です。この版にユーザー管理はありません。

## レジストリ側 Basic Auth

bcrypt パスワード ハッシュを含む htpasswd ファイルを作成します。次の例では、一時的な公式 Apache コンテナを使用します。認証情報の例を置き換えて、結果のファイルを保護します。

```sh
mkdir -p auth
docker run --rm --entrypoint htpasswd httpd:2   -Bbn your-user choose-a-strong-password > auth/htpasswd
```

Distribution 設定の例:

```yaml
auth:
  htpasswd:
    realm: Registry
    path: /auth/htpasswd
```

生成されたファイルを `/auth/htpasswd` のレジストリ コンテナーにマウントします。レジストリ構成は、Crateview 構成とは別のものです。リモート アクセスには HTTPS を使用します。 Basic Auth はエンコーディングであり、トランスポート暗号化ではありません。共有シェル履歴や公開サンプルに実際のパスワードを残さないようにします。

## UIセッション

Nitro サーバーは、レジストリと照合して資格情報を検証します。これらは、暗号化された HttpOnly、SameSite=Strict Cookie に保存されます。ブラウザーの JavaScript と localStorage は認証情報を受け取りません。セッションは 8 時間後に期限切れになります。

ランダムな本番セッションのシークレットを設定します。

```sh
openssl rand -base64 32
```

生成された値を `NUXT_SESSION_PASSWORD` として使用し、少なくとも 32 文字を使用します。ロード バランサーの背後にあるすべてのフロントエンド インスタンスは同じ値を使用する必要があります。構成を行わないと、Crateview はプロセス ローカル シークレットを生成するため、再起動すると既存のセッションが無効になります。

## HTTPS とログアウト

パブリック リクエスト プロトコルが HTTPS の場合、セッション Cookie はセキュアを使用します。リバース プロキシの転送プロトコルを正しく構成します。サインアウトすると UI セッションがクリアされます。レジストリ ユーザーは削除または変更されません。

Docker およびその他の CLI クライアントは、通常のレジストリ資格情報を使用して、`/v2/**` プロキシを通じて直接認証します。 Web UI にサインインしても、Docker CLI には自動的にサインインされません。
