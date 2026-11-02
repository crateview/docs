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


テーブルでタグを一つ以上選び、削除を確認します。プレビューは影響するタグを再取得し、同一ダイジェストは一度だけ削除し、部分失敗を報告します。すべての別名が影響を受けます。ガベージコレクションは別の管理作業です。

## ガベージコレクションとバックアップ

マニフェストを削除しても、すぐにディスクが再利用されるとは限りません。ガベージ コレクションは別個のレジストリ管理タスクです。 Crateview では実行されません。

ストレージ ドライバーに適した方法を使用して、レジストリ ストレージとその構成をバックアップします。容器を交換するときは容量を維持してください。ガベージ コレクションと書き込み調整については、Distribution の指示に従ってください。

- [Distribution ガベージ コレクション](https://distribution.github.io/distribution/about/garbage-collection/)
- [Distribution構成](https://distribution.github.io/distribution/about/configuration/)]

## Crateviewを更新

<!-- release:operations:start -->
プレビューコンテナは固定ソースからローカルビルドします。公開済みプレビューイメージは前提にしません。

```sh
git clone https://github.com/crateview/registry-ui.git
cd registry-ui
git checkout 9954f9c33e34c65bab19378a9b488e9b8ed13b6d
docker compose -f compose.yaml -f compose.build.yaml up --build -d
```
<!-- release:operations:end -->

## 大規模なレジストリ

リポジトリ検索では、ページ分割された完全なカタログが読み取られます。表示可能なリポジトリのメタデータはオンデマンドでフェッチされます。大規模なタグ リストをイメージの作成時間ごとに並べ替えるには、多くのマニフェストと構成を読み取る必要がある場合があります。 Crateview には、バックグラウンド メタデータ インデックスや外部検索サービスがありません。
