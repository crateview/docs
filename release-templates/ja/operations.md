リリース ノートを確認し、`CRATEVIEW_VERSION` を `.env` 内の目的の `v` プレフィックスが付いたリリース タグに設定し、フロントエンド コンテナーを置き換えます。

```sh
docker compose pull frontend
docker compose up -d --no-deps frontend
```

たとえば、初期リリースでは `CRATEVIEW_VERSION={{RELEASE_TAG}}` が使用されます。ロールバックするには、前のタグを復元し、これらのコマンドを繰り返します。ランタイム環境と `registry-data` ボリュームを保持します。 `docker compose down --volumes` は実行しないでください。ソース ビルドは、`compose.build.yaml` を通じて引き続き利用できます。

dependabot は、依存関係の更新を提案するようにソース リポジトリで構成されています。セルフホスト型の展開は自動的にアップグレードされません。アップグレード後に、レジストリの接続、サインイン、および影響を受けるアクションを再テストします。
