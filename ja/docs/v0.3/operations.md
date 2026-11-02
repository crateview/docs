---
title: "操作と削除"
description: "マニフェストを慎重に管理し、レジストリ ストレージを管理下に置きます。"
---

## マニフェストの削除を有効にする

Crateview UI では、削除はデフォルトで無効になっています。フロントエンドと Distribution の両方でこれを許可する必要があります。

```dotenv
NUXT_REGISTRY_DELETE_ENABLED=true
```

```yaml
storage:
  delete:
    enabled: true
```

Distribution は `REGISTRY_STORAGE_DELETE_ENABLED=true` とも読み取れます。環境または構成を変更した後、関連するサービスを再起動します。レジストリは、`/v2/**` を通じて直接行われたリクエストを含め、引き続き承認を担当します。

## digest の削除を理解する

標準の削除操作は、**マニフェスト digest** をターゲットとします。 digest を指すすべてのタグは削除できます。 Crateview は確認前に影響を受けるタグをプレビューしますが、同時ライターはプレビュー後に参照を変更できます。レジストリ API はエイリアスをロックするトランザクションを提供しません。

削除は、1 つのラベルの削除としてではなく、マニフェストとそのすべてのエイリアスに対する変更として扱います。確認する前に、バックアップと対象範囲を確認してください。

## ガベージコレクションとバックアップ

マニフェストを削除しても、すぐにディスクが再利用されるとは限りません。ガベージ コレクションは別個のレジストリ管理タスクです。 Crateview では実行されません。

ストレージ ドライバーに適した方法を使用して、レジストリ ストレージとその構成をバックアップします。容器を交換するときは容量を維持してください。ガベージ コレクションと書き込み調整については、Distribution の指示に従ってください。

- [Distribution ガベージ コレクション](https://distribution.github.io/distribution/about/garbage-collection/)
- [Distribution構成](https://distribution.github.io/distribution/about/configuration/)]

## Crateviewを更新

リリース ノートを確認し、`CRATEVIEW_VERSION` を `.env` 内の目的の `v` プレフィックスが付いたリリース タグに設定し、フロントエンド コンテナーを置き換えます。

```sh
docker compose pull frontend
docker compose up -d --no-deps frontend
```

たとえば、初期リリースでは `CRATEVIEW_VERSION=v0.3.0` が使用されます。ロールバックするには、前のタグを復元し、これらのコマンドを繰り返します。ランタイム環境と `registry-data` ボリュームを保持します。 `docker compose down --volumes` は実行しないでください。ソース ビルドは、`compose.build.yaml` を通じて引き続き利用できます。

dependabot は、依存関係の更新を提案するようにソース リポジトリで構成されています。セルフホスト型の展開は自動的にアップグレードされません。アップグレード後に、レジストリの接続、サインイン、および影響を受けるアクションを再テストします。

## 大規模なレジストリ

リポジトリ検索では、ページ分割された完全なカタログが読み取られます。表示可能なリポジトリのメタデータはオンデマンドでフェッチされます。大規模なタグ リストをイメージの作成時間ごとに並べ替えるには、多くのマニフェストと構成を読み取る必要がある場合があります。 Crateview には、バックグラウンド メタデータ インデックスや外部検索サービスがありません。
