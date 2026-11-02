---
title: "認証"
description: "匿名アクセス、Basic Auth、暗号化セッション、HTTPS。"
---

## サポートされている認証

Crateview は、匿名レジストリ、ネイティブ htpasswd Basic Auth、およびトークン モードのオプションの Crateview auth サービスをサポートします。 `/v2/` を調査し、レジストリで基本認証が必要な場合にサインイン フォームを表示します。

外部の Bearer トークン サービスと OIDC は統合されていません。未構成の外部トークン サービスでは、明示的な unsupported-auth 通知が生成されます。

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

## オプションのアカウント管理

独立した auth/管理サービスは、アカウントの作成と独自のパスワードの変更をサポートします。 Docker はレジストリに直接接続し、UI コンテナなしで動作します。ユーザーと bcrypt ハッシュは `htpasswd` に残ります。トークンの権限は `permissions.json` にあります。アカウント ストレージは SQLite を使用しません。

- **基本モード:** Distribution は共有 htpasswd ファイルをチェックします。すべてのユーザーが同等のレジストリ アクセス権を持ちます。 Docker では、管理サービスを実行する必要はありません。
- **トークン モード:** 独立した auth サービスは、署名された 5 分間のトークンを発行します。 Distribution はリポジトリ権限を強制します。 Docker には、レジストリと auth サービスが必要です。新しいユーザーには、最初はリポジトリへのアクセス権がありません。
- **UI:** サーバーの許可リストがアカウントの一覧表示、作成、パスワードリセット、削除を制御します。 `NUXT_USER_CREATION_ALLOWED_USERS`: `alice,bob`; `*`. 空値は管理を拒否し、列挙されたユーザー名は許可され、ワイルドカードは認証済みの全ユーザーを許可します。
- **権限:** CLI は、ターゲットそのもののリポジトリ、名前空間 (`team/*`)、またはすべて (`*`) を付与します。プロファイルは、`reader` (pull)、`writer` (pull/push)、および `manager` (pull/push/削除) です。基本モードにはレジストリの役割はありません。

`NUXT_AUTH_MODE=htpasswd` または `token` を明示的に有効にし、内部 `NUXT_AUTH_SERVICE_URL` およびサーバー専用 `NUXT_AUTH_SERVICE_SECRET` を構成します。トークン モードでは、`NUXT_AUTH_TOKEN_REALM` がパブリック トークン エンドポイントと正確に一致する必要があります。匿名レジストリと外部トークン サービスは、ローカル アカウント管理をサポートしていません。リポジトリ権限は認証 CLI で管理し、UI では読み取り専用で表示します。

Compose オーバーレイ、セットアップ、移行、および CLI コマンドについては、[アカウント サービス ガイド ](https://github.com/crateview/registry-ui/blob/3844d7605dac80d0d58999201c43e94c644d72c0/auth/README.md)] を参照してください。パブリック Docker アクセスを UI とは別にレジストリに直接ルーティングします。以前に発行されたトークンは、パスワードの変更または権限の取り消し後も最大 5 分間有効です。

管理には設定とユーザーがあります。アカウントメニューでも言語とテーマを選べます。アカウントでは現在のパスワードを確認して自分のパスワードを変更します。ユーザー一覧と作成はサーバー許可リストで制御され、トークン権限は読み取り専用です。新しいパスワードは12文字以上72 UTF-8バイト以下です。新しいトークンアカウントには最初リポジトリ権限がありません。

サーバーの許可リストがアカウントの一覧表示、作成、パスワードリセット、削除を制御します。 右端の三点メニューから、管理者は他のユーザーのパスワードを古いパスワードなしでリセットしたり、確認後にアカウントを削除したりできます。リセットではリポジトリ権限を保持し、削除では権限も削除します。自分のメニューは「アカウント」を開き、現在のパスワードで変更を確認します。自分のアカウントは削除できません。

![ドキュメントのバージョン](https://crateview.de/images/docs/next/users-light.webp)

![アクション](https://crateview.de/images/docs/next/user-actions-light.webp)

![パスワードを変更する](https://crateview.de/images/docs/next/user-password-light.webp)

![ユーザーを削除](https://crateview.de/images/docs/next/user-delete-light.webp)

```dotenv
NUXT_AUTH_MODE=htpasswd
NUXT_AUTH_SERVICE_URL=http://auth:8080
NUXT_AUTH_SERVICE_SECRET=REPLACE_WITH_SEPARATE_MANAGEMENT_SECRET
NUXT_USER_CREATION_ALLOWED_USERS=alice,bob
```

![ドキュメントのバージョン](https://crateview.de/images/docs/next/account-light.webp)

![ドキュメントのバージョン](https://crateview.de/images/docs/next/login-light.webp)
