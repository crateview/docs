---
title: "Otentikasi"
description: "Akses anonim, Basic Auth, sesi terenkripsi dan HTTPS."
---

## Otentikasi yang didukung

Crateview mendukung pendaftar anonim, htpasswd Basic Auth asli, dan layanan opsional Crateview auth dalam mode token. Ini menyelidiki `/v2/` dan menampilkan formulir masuk ketika registri memerlukan otentikasi dasar.

Layanan token Bearer eksternal dan OIDC tidak terintegrasi. Layanan token eksternal yang tidak dikonfigurasi menghasilkan pemberitahuan auth yang tidak didukung secara eksplisit.

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

## Manajemen akun opsional

Layanan auth/manajemen independen mendukung pembuatan akun dan perubahan kata sandi sendiri. Docker terhubung langsung ke registri dan berfungsi tanpa wadah UI. Pengguna dan hash bcrypt tetap berada di `htpasswd`; izin token ada di `permissions.json`. Penyimpanan akun tidak menggunakan SQLite.

- **Mode dasar:** Distribution memeriksa file htpasswd yang dibagikan; semua pengguna memiliki akses registri yang sama. Docker tidak memerlukan layanan manajemen untuk dijalankan.
- **Mode Token:** Layanan independen auth mengeluarkan token lima menit yang ditandatangani; Distribution menerapkan izin repositori. Docker memerlukan registri dan layanan auth. Pengguna baru awalnya tidak memiliki akses repositori.
- **UI:** Pengguna mengubah kata sandi mereka sendiri dengan memberikan kata sandi saat ini. `NUXT_USER_CREATION_ALLOWED_USERS` defaultnya kosong; `alice,bob` hanya mengizinkan pengguna tersebut untuk membuat akun, `*` mengizinkan setiap pengguna yang diautentikasi. Kebijakan ini diterapkan di sisi server.
- **Izin:** CLI memberikan target repositori yang tepat, namespace (`team/*`) atau semuanya (`*`); profilnya adalah `reader` (pull), `writer` (pull/push), dan `manager` (pull/push/hapus). Mode dasar tidak memiliki peran registri.

Aktifkan `NUXT_AUTH_MODE=htpasswd` atau `token` secara eksplisit dan konfigurasikan `NUXT_AUTH_SERVICE_URL` internal dan `NUXT_AUTH_SERVICE_SECRET` khusus server. Mode token juga memerlukan `NUXT_AUTH_TOKEN_REALM` agar sama persis dengan titik akhir token publik. Registri anonim dan layanan token eksternal tidak mendukung pengelolaan akun lokal. Izin dan pengaturan ulang kata sandi administratif menggunakan auth CLI; UI tidak dapat mengubah kata sandi pengguna lain atau memberikan izin.

Lihat [panduan layanan akun](https://github.com/crateview/registry-ui/blob/main/auth/README.md) untuk perintah Compose overlay, setup, migrasi, dan CLI. Rutekan akses publik Docker langsung ke registri secara independen dari UI. Token yang dikeluarkan sebelumnya tetap valid hingga lima menit setelah perubahan kata sandi atau pencabutan izin.
