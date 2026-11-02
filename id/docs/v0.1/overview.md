---
title: "Ringkasan"
description: "Apa yang dilakukan Crateview, bagaimana ia terhubung ke registri Anda, dan di mana batasannya."
---

## Tampilan registri Anda yang lebih jelas

Crateview adalah antarmuka web untuk registrasi kontainer Docker dan OCI. Ini membantu Anda menemukan repositori dan tag, memeriksa gambar multi-platform, membaca manifes dan mengonfigurasi dokumen, serta memahami lapisan terkompresi dan riwayatnya.

Ini berjalan bersama registri Anda. Registri menyimpan gambar dan menerapkan kontrol akses; Crateview menyediakan antarmuka dan proxy asal yang sama. Docker dan klien OCI lainnya dapat menggunakan asal frontend sebagai alamat registri mereka.

![Ikhtisar repositori Crateview dengan Alpine dan repositori demo bersarang.](https://crateview.de/images/docs/v0.1/repositories-light.webp)

## Arsitektur

```text
Browser / Docker / Podman / ORAS
              |
       Crateview (Nuxt / Nitro)
              |
   Your existing OCI registry
```

Frontend membaca Registry HTTP API V2 standar. Metadata UI berasal dari katalog, daftar tag, manifes, dan blob konfigurasi gambar. Itu tidak memerlukan indeks pencarian atau database aplikasi terpisah.

## Fitur yang didukung

- Pencarian nama repositori, saran tag, dan ruang nama bersarang.
- Tabel tag, pemilihan platform, ukuran lapisan terkompresi, dan riwayat gambar.
- Anotasi OCI untuk deskripsi, URL sumber, dokumentasi, dan lisensi.
- Manifes mentah dan konfigurasi JSON dengan dukungan clipboard.
- Perujuk OCI, termasuk fallback indeks tag digest standar yang dikelola oleh klien OCI.
- Akses anonim dan htpasswd Basic Auth asli.
- Penghapusan manifes opsional, dengan tag yang terpengaruh ditampilkan sebelum konfirmasi.
- Dua bahasa antarmuka (Inggris dan Jerman), Sistem/Terang/Gelap dan merek yang dapat diatur.


## Ketahui batasannya

API standar tidak menyediakan stempel waktu push, statistik unduhan, favorit, atau pemindaian kerentanan. Crateview tidak membuat nilai-nilai ini. Tanggal pembuatan gambar berasal dari metadata gambar dan bukan waktu pengunggahannya. Ukuran yang ditampilkan menjelaskan lapisan gambar terkompresi, bukan penggunaan disk registri.

Layanan token Bearer eksternal, OIDC, administrasi pengguna, dan manajemen kunci API tidak terintegrasi. Artefak tanda tangan dapat dicantumkan sebagai perujuk; ini bukan verifikasi tanda tangan kriptografi.

Mulailah dengan [installation](/id/docs/v0.1/installation), lalu tinjau [configuration](/id/docs/v0.1/configuration) dan [authentication](/id/docs/v0.1/authentication).
