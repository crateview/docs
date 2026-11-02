---
title: "監視"
description: "監視 · Distribution · SQLite"
---

## 監視

監視は任意で初期状態は無効です。有効時は概要がダッシュボードを開き、無効時は /home が /explore に転送されます。有効化すると公開レジストリの匿名訪問者を含む全UI利用者にレジストリ全体の運用情報が公開されます。管理トークンモードでは全体の読み取り権限も必要です。

![監視](https://crateview.de/images/docs/next/monitoring-light.webp)

## この版の設定

専用の Webhook シークレットを生成し、対応するソースから以下のオーバーレイを起動します。Compose 2.23+ が必要です。独自認証や保存設定は付属のファイルシステム設定と統合します。デバッグ指標は内部限定にします。SQLite は永続化されるため、更新前に WAL を含む親ディレクトリをバックアップします。

```sh
openssl rand -hex 32
```

```dotenv
NUXT_MONITORING_WEBHOOK_SECRET=REPLACE_WITH_GENERATED_WEBHOOK_SECRET
```

```sh
docker compose -f compose.yaml -f compose.monitoring.yaml -f compose.build.yaml up --build -d
```

| ENV | Default |
| --- | --- |
| `NUXT_MONITORING_ENABLED` | `false` |
| `NUXT_MONITORING_METRICS_URL` | — |
| `NUXT_MONITORING_METRICS_TOKEN` | — |
| `NUXT_MONITORING_WEBHOOK_SECRET` | — |
| `NUXT_MONITORING_DATABASE_PATH` | `.dev/monitoring.sqlite` |

```yaml
# Distribution: merge with your existing configuration.
http:
  addr: :5000
  debug:
    addr: :5001
    prometheus:
      enabled: true
notifications:
  endpoints:
    - name: crateview
      url: http://frontend:3000/api/monitoring/events
      headers:
        Authorization: ["Bearer REPLACE_WITH_GENERATED_WEBHOOK_SECRET"]
      timeout: 5s
      threshold: 5
      backoff: 1s
```

```dotenv
# Existing Distribution: server-only configuration, internal metrics endpoint.
NUXT_MONITORING_ENABLED=true
NUXT_MONITORING_METRICS_URL=http://registry:5001/metrics
NUXT_MONITORING_METRICS_TOKEN=
NUXT_MONITORING_WEBHOOK_SECRET=REPLACE_WITH_GENERATED_WEBHOOK_SECRET
NUXT_MONITORING_DATABASE_PATH=/data/monitoring.sqlite
```

## 24h / 7d

この版の設定. Node.js 26; Distribution; SQLite.

24時間と7日間の表示、60秒ごとの収集、速度には2回の計測が必要です。保存期間は30日です。再起動、カウンタリセット、欠損は補間せず空白になります。通知に過去の補完はありません。マニフェストイベントは完全なダウンロード数ではなく、保存操作はディスク使用量ではありません。1プロセスと1 Distribution をサポートします。401の検査結果は認証が必要だが到達可能という意味です。ネットワーク/5xx障害時、確認済みセッションは期限まで読めますが新規認証はできません。グラフがない場合は内部指標、Webhook Bearer シークレット、SQLite書き込み権限、2回の計測を確認してください。
