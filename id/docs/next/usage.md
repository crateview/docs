---
title: "Menggunakan Crateview"
description: "Navigasikan gambar, ganti platform, dan periksa data di balik tag."
---

## Temukan repositori dan tag

Jelajah menampilkan repositori. Pencarian mendukung nama dan repository:tag. Buka repositori untuk melihat tag.

Repositori memakai tabel secara default; tersedia tampilan kisi. Nama dengan garis miring membentuk namespace. Bilah samping dapat diubah ukurannya dan diciutkan, menyimpan keadaan lokal dan memuat pencarian.

/ atau Ctrl+K (Cmd+K di macOS) membuka palet pencarian dan navigasi. Folder namespace dapat diciutkan. Pengaturan memuat bahasa, Sistem/Terang/Gelap dan Tentang dengan lisensi dependensi serta pemberitahuan asli yang dapat dicari.

Administrasi mengelompokkan Pengaturan dan Pengguna. Menu akun juga menawarkan bahasa dan tema. Akun mengubah kata sandi sendiri setelah memverifikasi yang sekarang. Daftar izin server mengontrol daftar dan pembuatan akun; izin token hanya baca. Kata sandi baru minimal 12 karakter dan maksimal 72 byte UTF-8. Akun token baru belum memiliki akses repositori.

Daftar izin server mengontrol daftar, pembuatan, pengaturan ulang kata sandi, dan penghapusan akun. Menu tiga titik di sebelah kanan memungkinkan administrator mengatur ulang kata sandi pengguna lain tanpa kata sandi lama atau menghapus akun tersebut setelah konfirmasi. Pengaturan ulang mempertahankan izin repositori; penghapusan menghapusnya. Menu akun Anda sendiri membuka Akun untuk mengubah kata sandi dengan kata sandi saat ini; penghapusan akun sendiri diblokir.

![Pencitraan merek & tema](https://crateview.de/images/docs/next/settings-light.webp)

## Periksa gambar

Buka repositori dan pilih tag. Tampilan gambar menunjukkan digest, platform yang tersedia, ukuran lapisan terkompresi, dan metadata pembuatan.

Untuk gambar multi-platform, pilih sistem operasi dan arsitektur di pemilih platform. Riwayat lapisan dan konfigurasi gambar yang dipilih mengikuti pilihan itu. Tab Manifest mentah menampilkan manifes atau indeks akar gambar; Konfigurasi menunjukkan konfigurasi platform yang dipilih.

![Detail gambar Alpen dengan pemilihan platform, riwayat lapisan, dan perintah pull.](https://crateview.de/images/docs/next/image-light.webp)

## Perintah Pull

Panel perintah menyediakan perintah Docker, Podman dan Skopeo untuk gambar, dan perintah artefak ORAS/Podman untuk artefak. Salin perintah dan jalankan di terminal Anda sendiri.

```sh
docker pull registry.example.com/team/service:latest
podman pull registry.example.com/team/service:latest
oras pull registry.example.com/team/report:latest
```

Ganti host contoh dengan asal Crateview publik Anda. Otentikasi CLI terpisah dari sesi browser Anda.

## Metadata dan perujuk OCI

Deskripsi, lisensi, dan tautan sumber dibaca dari anotasi OCI atau label konfigurasi gambar bila tersedia. Metadata yang hilang akan tetap kosong dan tidak dapat ditebak.

Tab Perujuk membaca perujuk OCI API. Jika registri tidak mengimplementasikan titik akhir tersebut, Crateview membaca indeks tag turunan digest standar yang dikelola oleh klien OCI. Itu tidak membangun indeksnya sendiri dengan memindai setiap manifes. Artefak tanda tangan yang tercantum tidak secara otomatis diverifikasi atau ditandai sebagai tepercaya.

## Menafsirkan stempel waktu dan ukuran

Waktu pembuatan gambar disediakan oleh konfigurasi gambar atau anotasi. Itu tidak memberi tahu Anda kapan tag dimasukkan ke registri ini. Ukuran lapisan terkompresi tidak mengukur konsumsi disk, karena lapisan dapat dibagikan dan data yang dihapus mungkin tetap ada hingga pengumpulan sampah.
