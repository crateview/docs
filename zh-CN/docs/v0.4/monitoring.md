---
title: "监控"
description: "监控 · Distribution · SQLite"
---

## 监控

监控可选，默认关闭。启用后概览显示仪表盘，否则 /home 重定向至 /explore。启用会向所有有界面访问权的用户开放整个注册表的运行数据，公共注册表也包括匿名访问者。

![监控](https://crateview.de/images/docs/v0.4/monitoring-light.webp)

## 此版本的配置

生成独立的 Webhook 密钥，然后从匹配源码运行以下覆盖配置。要求 Compose 2.23+。使用自定义认证或存储时需合并随附的文件系统配置。调试指标仅限内部访问。覆盖配置持久化 SQLite；升级前备份其父目录及 WAL 文件。

```sh
openssl rand -hex 32
```

```dotenv
CRATEVIEW_VERSION=v0.4.0
NUXT_MONITORING_WEBHOOK_SECRET=REPLACE_WITH_GENERATED_WEBHOOK_SECRET
```

```sh
docker compose -f compose.yaml -f compose.monitoring.yaml up -d
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

此版本的配置. Node.js 24; Distribution; SQLite.

仪表盘提供24小时和7天窗口，每60秒采样，速率需要两次采样。保留30天。重启建立新基线；中断、计数器重置和缺失指标显示为间隙，不插补历史。通知不回填历史。清单事件不是完整下载，存储操作指标不是磁盘使用量。仅支持一个服务进程和一个 Distribution 实例。401探测表示可达但需要认证。网络或5xx故障期间，已验证会话可读取直至过期，新会话无法授权。图表缺失时检查内部指标、Webhook Bearer 密钥、SQLite写权限及两次完整采样。
