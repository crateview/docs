---
title: "Operasi & penghapusan"
description: "Kelola manifes dengan hati-hati dan simpan penyimpanan registri di bawah kendali Anda."
---

## Aktifkan penghapusan manifes

Penghapusan dinonaktifkan di UI Crateview secara default. Frontend dan Distribution harus mengizinkannya:

```dotenv
NUXT_REGISTRY_DELETE_ENABLED=true
```

```yaml
storage:
  delete:
    enabled: true
```

Distribution juga dapat membaca `REGISTRY_STORAGE_DELETE_ENABLED=true`. Mulai ulang layanan yang relevan setelah mengubah lingkungan atau konfigurasinya. Registri tetap bertanggung jawab atas otorisasi, termasuk permintaan yang dibuat langsung melalui `/v2/**`.

## Pahami penghapusan digest

Operasi penghapusan standar menargetkan **manifes digest**. Setiap tag yang menunjuk ke digest dapat dihapus. Crateview mempratinjau tag yang terpengaruh sebelum konfirmasi, namun penulis secara bersamaan dapat mengubah referensi setelah pratinjau: registri API tidak menyediakan transaksi yang mengunci alias.

Perlakukan penghapusan sebagai perubahan pada manifes dan semua aliasnya, bukan sebagai penghapusan satu label. Verifikasi cadangan Anda dan cakupan yang diinginkan sebelum mengonfirmasi.


Pilih satu atau beberapa tag di tabel dan konfirmasi penghapusan. Pratinjau memperbarui tag terdampak, menghapus setiap digest sekali dan melaporkan kegagalan parsial. Semua alias terdampak. Pengumpulan sampah tetap tugas registri terpisah.

## Pengumpulan dan pencadangan sampah

Menghapus manifes tidak menjanjikan reklamasi disk secara langsung. Pengumpulan sampah adalah tugas administrasi pencatatan yang terpisah; Crateview tidak menjalankannya.

Cadangkan penyimpanan registri Anda dan konfigurasinya menggunakan metode yang sesuai dengan driver penyimpanan. Pertahankan volume saat mengganti wadah. Ikuti instruksi Distribution untuk pengumpulan sampah dan tulis koordinasi:

- [Distribution pengumpulan sampah](https://distribution.github.io/distribution/about/garbage-collection/)
- [Konfigurasi Distribution](https://distribution.github.io/distribution/about/configuration/)

## Perbarui Crateview

Tinjau catatan rilis, atur `CRATEVIEW_VERSION` ke tag rilis dengan awalan `v` yang diinginkan di `.env`, dan ganti penampung frontend:

```sh
docker compose pull frontend
docker compose up -d --no-deps frontend
```

Misalnya, rilis awal menggunakan `CRATEVIEW_VERSION=v0.8.0`. Untuk memutar kembali, pulihkan tag sebelumnya dan ulangi perintah ini. Pertahankan lingkungan runtime dan volume `registry-data`; jangan jalankan `docker compose down --volumes`. Pembuatan sumber tetap tersedia melalui `compose.build.yaml`.

Dependabot dikonfigurasikan di repositori sumber untuk mengusulkan pembaruan ketergantungan; itu tidak meningkatkan penerapan yang dihosting sendiri secara otomatis. Uji ulang konektivitas registri, proses masuk, dan tindakan yang terpengaruh setelah peningkatan.

## Registri besar

Pencarian repositori membaca katalog paginasi lengkap. Metadata untuk repositori yang terlihat diambil berdasarkan permintaan. Mengurutkan daftar tag besar berdasarkan waktu pembuatan gambar memerlukan pembacaan banyak manifes dan konfigurasi. Crateview tidak memiliki indeks metadata latar belakang atau layanan pencarian eksternal.
