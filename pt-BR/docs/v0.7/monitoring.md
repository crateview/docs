---
title: "Monitoramento"
description: "Monitoramento · Distribution · SQLite"
---

## Monitoramento

Monitoramento opcional, desativado por padrão. Visão geral abre o painel quando ativo; caso contrário /home redireciona para /explore. A ativação expõe dados globais a todos com acesso à interface, incluindo visitantes anônimos em registros públicos. O modo token gerenciado exige leitura global adicional.

![Monitoramento](https://crateview.de/images/docs/v0.7/monitoring-light.webp)

## Configuração desta versão

Gere um segredo Webhook separado e execute a sobreposição da revisão correspondente. Requer Compose 2.23+. Integre sua configuração de sistema de arquivos à autenticação ou armazenamento personalizados. Mantenha métricas de depuração internas. SQLite é persistente; faça backup do diretório pai e arquivos WAL antes de atualizar.

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

Configuração desta versão. Node.js 26; Distribution; SQLite.

Janelas de 24 horas e 7 dias; amostras a cada 60 segundos; taxas exigem duas amostras. Retenção de 30 dias. Reinícios, contadores reiniciados e métricas ausentes produzem lacunas sem interpolação. Notificações não preenchem histórico. Eventos de manifesto não são downloads completos; operações de armazenamento não medem disco. Suporta um processo e uma instância Distribution. Sonda 401 indica acessível com autenticação necessária. Em falhas de rede/5xx, sessões verificadas leem até expirar; novas sessões não são autorizadas. Sem gráficos, verifique métricas internas, segredo Bearer Webhook, escrita SQLite e duas amostras completas.
