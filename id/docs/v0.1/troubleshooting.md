---
title: "Pemecahan masalah"
description: "Selesaikan masalah koneksi, autentikasi, metadata, dan penerapan."
---

## UI tidak dapat menjangkau registri

Periksa asal registri dari lingkungan jaringan server frontend. Dalam sebuah container, `localhost` menunjuk ke container itu sendiri. Upstream Compose yang umum adalah `http://registry:5000`.

```sh
curl -i http://127.0.0.1:5000/v2/
```

Registri anonim akan mengembalikan kesuksesan. Yang diautentikasi dapat mengembalikan `401` dengan tantangan `WWW-Authenticate`. Penolakan koneksi berarti pendengar, alamat atau jalur jaringan belum siap.

## Gagal masuk

Konfirmasikan bahwa registri menggunakan htpasswd Basic Auth asli dan file bcrypt-nya dipasang dengan benar. Tantangan token Bearer eksternal tidak didukung oleh UI saat ini.

Untuk kehilangan sesi berulang, periksa apakah semua instans frontend berbagi `NUXT_SESSION_PASSWORD`. Rahasia yang dihasilkan oleh proses lokal berubah setelah restart. Verifikasi juga HTTPS dan header protokol yang diteruskan; sesi browser memerlukan cookie untuk kembali ke asal yang sama.

## Gambar saya tidak memiliki deskripsi atau tanggal pembuatan

Nilai-nilai ini harus ada dalam anotasi manifes atau konfigurasi gambar. Crateview tidak menyimpulkan metadata yang hilang atau waktu push registri pengganti. Tambahkan anotasi OCI yang sesuai saat membuat gambar Anda.

## Perujuk kosong

Registri perlu mengekspos titik akhir perujuk OCI atau indeks fallback tag digest standar. Klien OCI bertanggung jawab untuk memelihara indeks cadangan tersebut bila diperlukan. Hanya memasukkan manifes yang memuat subjek ke dalam registri tanpa dukungan perujuk tidak membuat Crateview memindai semua tag untuk manifes tersebut.

## Penghapusan tidak ada atau ditolak

Aktifkan keduanya Crateview'S `NUXT_REGISTRY_DELETE_ENABLED` Dan Distributionpengaturan penghapusan penyimpanan. Registri masih dapat menolak permintaan tersebut sesuai dengan izin atau implementasinya. Penghapusan berhasil digest dan dapat memengaruhi banyak tag.

## Produksi mengabaikan file .env saya

`node .output/server/index.mjs` tidak secara otomatis memuat `.env`. Ekspor variabel melalui manajer proses, lingkungan kontainer, atau shell Anda. Mulai ulang setelah mengubah nilai runtime. Host web statis saja tidak dapat menggantikan server Nitro yang diperlukan.

## Perintah pull yang disalin tidak berfungsi

Perintah ini menggunakan alamat frontend publik. Konfirmasikan bahwa CLI dapat mencapai asal tersebut dan memercayai sertifikat TLS-nya. Otentikasi CLI secara terpisah bila diperlukan. Masuk browser tidak membuat login Docker CLI.

Jika Anda menemukan masalah UI yang dapat direproduksi, buka [issue](https://github.com/crateview/registry-ui/issues) dengan revisi yang terpengaruh dan detail reproduksi yang telah dibersihkan. Gunakan [saluran keamanan pribadi](https://github.com/crateview/registry-ui/security/policy) untuk mengetahui kerentanan.
