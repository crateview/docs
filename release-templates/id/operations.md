Tinjau catatan rilis, atur `CRATEVIEW_VERSION` ke tag rilis dengan awalan `v` yang diinginkan di `.env`, dan ganti penampung frontend:

```sh
docker compose pull frontend
docker compose up -d --no-deps frontend
```

Misalnya, rilis awal menggunakan `CRATEVIEW_VERSION={{RELEASE_TAG}}`. Untuk memutar kembali, pulihkan tag sebelumnya dan ulangi perintah ini. Pertahankan lingkungan runtime dan volume `registry-data`; jangan jalankan `docker compose down --volumes`. Pembuatan sumber tetap tersedia melalui `compose.build.yaml`.

Dependabot dikonfigurasikan di repositori sumber untuk mengusulkan pembaruan ketergantungan; itu tidak meningkatkan penerapan yang dihosting sendiri secara otomatis. Uji ulang konektivitas registri, proses masuk, dan tindakan yang terpengaruh setelah peningkatan.
