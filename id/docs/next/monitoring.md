---
title: "Pemantauan"
description: "Pemantauan · Distribution · SQLite"
---

## Pemantauan

Pemantauan opsional dan nonaktif secara default. Ringkasan membuka dasbor jika aktif; selain itu /home mengalihkan ke /explore. Aktivasi membuka data operasional global bagi semua pengakses UI, termasuk pengunjung anonim registri publik. Mode token terkelola juga memerlukan izin baca global.

![Pemantauan](https://crateview.de/images/docs/next/monitoring-light.webp)

## Konfigurasi versi ini

Buat rahasia Webhook terpisah lalu jalankan overlay dari revisi yang cocok. Perlu Compose 2.23+. Gabungkan konfigurasi sistem berkas dengan autentikasi atau penyimpanan khusus. Metrik debug harus internal. SQLite persisten; cadangkan direktori induk termasuk berkas WAL sebelum peningkatan.

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

Konfigurasi versi ini. Node.js 26; Distribution; SQLite.

Jendela 24 jam dan 7 hari; sampel setiap 60 detik; laju perlu dua sampel. Retensi 30 hari. Mulai ulang, reset penghitung dan metrik hilang membuat celah tanpa interpolasi. Notifikasi tidak mengisi riwayat lampau. Peristiwa manifest bukan unduhan lengkap; operasi penyimpanan bukan penggunaan disk. Mendukung satu proses dan satu instans Distribution. Probe 401 berarti terjangkau tetapi perlu autentikasi. Saat gangguan jaringan/5xx, sesi terverifikasi dapat membaca hingga kedaluwarsa; sesi baru tidak diotorisasi. Jika grafik hilang, periksa metrik internal, rahasia Bearer Webhook, izin tulis SQLite dan dua sampel lengkap.
