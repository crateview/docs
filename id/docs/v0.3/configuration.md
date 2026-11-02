---
title: "Konfigurasi"
description: "Koneksi registri, variabel lingkungan runtime, dan perilaku penerapan."
---

## Lapisan konfigurasi

Default aplikasi ditentukan dalam `nuxt.config.ts` dan `app/app.config.ts`. Variabel lingkungan menggantikan default runtime saat server dimulai. Perintah pengembangan dan pembuatan membaca `.env`; titik masuk Node produksi tidak membacanya secara otomatis.

Mulai ulang server setelah mengubah variabel lingkungan runtime. Mengedit konfigurasi komponen tingkat sumber memerlukan build produksi baru.

## Pengaturan registri dan sesi

| Variabel | Bawaan | Tujuan |
| ------------------------------ | ----------------------- | ----------------------------------- |
| `NUXT_REGISTRY_UPSTREAM` | `http://127.0.0.1:5000` | Asal registri sisi server. |
| `NUXT_REGISTRY_DELETE_ENABLED` | `false` | Ekspos penghapusan manifes di UI. |
| `NUXT_SESSION_PASSWORD` | Acak per proses | Rahasia yang digunakan untuk mengenkripsi sesi UI. |
| `HOST` | Bergantung pada waktu proses | Antarmuka mendengarkan server node. |
| `PORT` | `3000` | Port mendengarkan server node. |

Upstream harus berupa asal HTTP(S) yang dapat dijangkau dari server frontend. Jauhkan kredensial dari URL. Permintaan menggunakan otentikasi pengguna saat ini; Crateview tidak menggantikan kontrol akses registri.

## Variabel branding publik

Semua nama di bawah menggunakan awalan `NUXT_PUBLIC_BRAND_`:

| Akhiran | Tujuan |
| ------------------------------------------- | ----------------------------------------------- |
| `NAME` | Nama aplikasi. |
| `DESCRIPTION` | Deskripsi aplikasi. |
| `LOGO_LIGHT`, `LOGO_DARK` | Jalur logo atau URL HTTP(S). |
| `FAVICON` | Jalur atau URL ikon browser. |
| `TITLE_TEMPLATE` | Judul halaman; `%s` diganti dengan judul halaman. |
| `PRODUCT_URL`, `DOCS_URL`, `REPOSITORY_URL` | Tautan navigasi opsional. |

Konfigurasi publik dikirim ke browser. Jangan pernah menyimpan kata sandi atau token dalam variabel `NUXT_PUBLIC_*`. Lihat [pencitraan merek dan tema](/id/docs/v0.3/theming) untuk pilihan warna dan tipografi.

## Tata letak jaringan

Browser berkomunikasi dengan Crateview di satu asal. Nitro memproksi `/v2/**` untuk klien registri dan membaca metadata registri melalui `/api/registry/**`. Hal ini menghindari keharusan akses browser lintas asal ke registri itu sendiri.

Di balik proksi terbalik, pertahankan asal publik dan header protokol. Alamat frontend publik juga merupakan alamat registri yang ditampilkan dalam perintah pull yang disalin. Untuk penggunaan yang diautentikasi jarak jauh, hentikan HTTPS dengan benar sebelum menerima kredensial browser atau CLI.

## Compose versus konfigurasi mandiri

File Compose yang dibundel sengaja dihubungkan ke layanan `registry` miliknya sendiri dengan `NUXT_REGISTRY_UPSTREAM=http://registry:5000`. Untuk menggunakan registri lain dalam penerapan kontainer, sesuaikan lingkungan layanan tersebut atau jalankan frontend secara terpisah. Jangan berasumsi bahwa registri loopback host dapat dijangkau melalui gateway kontainer.
