---
title: "Otentikasi"
description: "Akses anonim, Basic Auth, sesi terenkripsi dan HTTPS."
---

## Otentikasi yang didukung

Mendukung akses anonim dan htpasswd Basic Auth bawaan. Layanan Bearer eksternal dan OIDC tidak didukung. Versi ini tidak mengelola pengguna.

## Basic Auth sisi registri

Buat file htpasswd dengan hash kata sandi bcrypt. Contoh berikut menggunakan wadah Apache resmi sementara; ganti kredensial contoh dan lindungi file yang dihasilkan:

```sh
mkdir -p auth
docker run --rm --entrypoint htpasswd httpd:2   -Bbn your-user choose-a-strong-password > auth/htpasswd
```

Contoh pengaturan Distribution:

```yaml
auth:
  htpasswd:
    realm: Registry
    path: /auth/htpasswd
```

Pasang file yang dihasilkan ke dalam wadah registri di `/auth/htpasswd`. Konfigurasi registri terpisah dari konfigurasi Crateview. Gunakan HTTPS untuk akses jarak jauh; Basic Auth adalah pengkodean, bukan enkripsi transport. Hindari meninggalkan kata sandi asli dalam riwayat shell bersama atau contoh publik.

## Sesi UI

Server Nitro memvalidasi kredensial terhadap registri. Ia menyimpannya dalam cookie terenkripsi, HttpOnly, SameSite=Strict. Browser JavaScript dan localStorage tidak menerima kredensial. Sesi berakhir setelah delapan jam.

Tetapkan rahasia sesi produksi acak:

```sh
openssl rand -base64 32
```

Gunakan nilai yang dihasilkan sebagai `NUXT_SESSION_PASSWORD`, dengan setidaknya 32 karakter. Setiap instance frontend di belakang penyeimbang beban harus menggunakan nilai yang sama. Tanpa konfigurasi, Crateview menghasilkan rahasia lokal proses, sehingga memulai ulang akan membatalkan sesi yang ada.

## HTTPS dan keluar

Cookie sesi menggunakan Aman ketika protokol permintaan publik adalah HTTPS. Konfigurasikan protokol penerusan proksi terbalik dengan benar. Keluar akan menghapus sesi UI; itu tidak menghapus atau mengubah pengguna registri.

Docker dan klien CLI lainnya mengautentikasi secara langsung melalui proksi `/v2/**` menggunakan kredensial registri normalnya. Masuk ke UI web tidak memasukkan Docker CLI secara otomatis.
