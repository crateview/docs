---
title: "Ringkasan"
description: "Apa yang dilakukan Crateview, bagaimana ia terhubung ke registri Anda, dan di mana batasannya."
---

## Tampilan registri Anda yang lebih jelas

Crateview adalah antarmuka web untuk registrasi kontainer Docker dan OCI. Ini membantu Anda menemukan repositori dan tag, memeriksa gambar multi-platform, membaca manifes dan mengonfigurasi dokumen, serta memahami lapisan terkompresi dan riwayatnya.

Ini berjalan bersama registri Anda. Registri menyimpan gambar dan menerapkan kontrol akses; Crateview menyediakan antarmuka dan proxy asal yang sama. Docker dan klien OCI lainnya dapat menggunakan asal frontend sebagai alamat registri mereka.

![Ikhtisar repositori Crateview dengan Alpine dan repositori demo bersarang.](https://crateview.de/images/docs/next/repositories-light.webp)

## Arsitektur

```text
Browser / Docker / Podman / ORAS
              |
       Crateview (Nuxt / Nitro)
              |
   Your existing OCI registry
```

Katalog tidak perlu basis data; pemantauan opsional menggunakan SQLite persisten. Akun memakai htpasswd dan berkas izin terpisah.

## Fitur yang didukung

- Pencarian nama repositori, saran tag, dan ruang nama bersarang.
- Tabel tag, pemilihan platform, ukuran lapisan terkompresi, dan riwayat gambar.
- Anotasi OCI untuk deskripsi, URL sumber, dokumentasi, dan lisensi.
- Manifes mentah dan konfigurasi JSON dengan dukungan clipboard.
- Perujuk OCI, termasuk fallback indeks tag digest standar yang dikelola oleh klien OCI.
- Akses anonim dan htpasswd Basic Auth asli.
- Penghapusan manifes opsional, dengan tag yang terpengaruh ditampilkan sebelum konfirmasi.
- 13 bahasa antarmuka, Sistem/Terang/Gelap, dan branding yang dapat dikonfigurasi.

- Pemantauan opsional dan nonaktif secara default. Ringkasan membuka dasbor jika aktif; selain itu /home mengalihkan ke /explore. Aktivasi membuka data operasional global bagi semua pengakses UI, termasuk pengunjung anonim registri publik. Mode token terkelola juga memerlukan izin baca global.

## Ketahui batasannya

API standar tidak menyediakan stempel waktu push, statistik unduhan, favorit, atau pemindaian kerentanan. Crateview tidak membuat nilai-nilai ini. Tanggal pembuatan gambar berasal dari metadata gambar dan bukan waktu pengunggahannya. Ukuran yang ditampilkan menjelaskan lapisan gambar terkompresi, bukan penggunaan disk registri.

Administrasi mengelompokkan Pengaturan dan Pengguna. Menu akun juga menawarkan bahasa dan tema. Akun mengubah kata sandi sendiri setelah memverifikasi yang sekarang. Daftar izin server mengontrol daftar dan pembuatan akun; izin token hanya baca. Kata sandi baru minimal 12 karakter dan maksimal 72 byte UTF-8. Akun token baru belum memiliki akses repositori.

Daftar izin server mengontrol daftar, pembuatan, pengaturan ulang kata sandi, dan penghapusan akun. Menu tiga titik di sebelah kanan memungkinkan administrator mengatur ulang kata sandi pengguna lain tanpa kata sandi lama atau menghapus akun tersebut setelah konfirmasi. Pengaturan ulang mempertahankan izin repositori; penghapusan menghapusnya. Menu akun Anda sendiri membuka Akun untuk mengubah kata sandi dengan kata sandi saat ini; penghapusan akun sendiri diblokir.

Mulailah dengan [installation](/id/docs/next/installation), lalu tinjau [configuration](/id/docs/next/configuration) dan [authentication](/id/docs/next/authentication).
