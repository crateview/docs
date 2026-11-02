---
title: "Pencitraan merek & tema"
description: "Gunakan identitas Anda sendiri sambil tetap mempertahankan sistem desain asli Nuxt UI."
---

![Pencitraan merek & tema](https://crateview.de/images/docs/next/settings-light.webp)

## Default

Crateview menggunakan palet utama `red` Nuxt UI dan palet netral `zinc`. Mode warna default adalah Sistem; pengguna dapat memilih Terang atau Gelap terlepas dari konfigurasi warna Anda.

## Variabel tema runtime

```dotenv
NUXT_PUBLIC_THEME_PRIMARY=red
NUXT_PUBLIC_THEME_NEUTRAL=zinc
NUXT_PUBLIC_THEME_RADIUS=0.5rem
NUXT_PUBLIC_THEME_FONT_FAMILY=Inter, ui-sans-serif, system-ui, sans-serif
```

Palet utama menerima warna aksen Tailwind standar seperti `blue`, `emerald`, atau `violet`. Netral menerima `slate`, `gray`, `zinc`, `neutral` atau `stone`. Radius menerima nilai `px` atau `rem` non-negatif. Konfigurasi font memilih tumpukan font; bundel file font dan nyatakan `@font-face` jika font Anda tidak diinstal pada klien.

Mulai ulang server setelah mengubah variabel lingkungan produksi. Tidak diperlukan pembangunan kembali untuk pengaturan runtime ini.

## Konfigurasi aplikasi dan komponen

Defaultnya ditentukan di `app/app.config.ts`:

```ts
export default defineAppConfig({
  ui: { colors: { primary: 'red', neutral: 'zinc' } },
  theme: {
    radius: '0.5rem',
    fontFamily: 'Inter, ui-sans-serif, system-ui, sans-serif',
  },
})
```

Gunakan konfigurasi Nuxt UI `ui` normal untuk varian komponen dan ikon. Token desain CSS tambahan dan aturan permukaan terang/gelap ada di `app/assets/css/main.css`. Perubahan sumber memerlukan pembangunan kembali aplikasi produksi.

## Ganti mereknya

```dotenv
NUXT_PUBLIC_BRAND_NAME=My Registry
NUXT_PUBLIC_BRAND_LOGO_LIGHT=/my-logo.png
NUXT_PUBLIC_BRAND_LOGO_DARK=/my-logo.png
NUXT_PUBLIC_BRAND_FAVICON=/my-favicon.ico
NUXT_PUBLIC_BRAND_TITLE_TEMPLATE=%s · My Registry
```

Tempatkan aset lokal di `public/` sebelum membangun, atau gunakan URL aset HTTP(S). Tautan produk, dokumentasi, dan repositori opsional yang ada dapat dikonfigurasi melalui variabel `NUXT_PUBLIC_BRAND_*_URL` yang sesuai. Konfigurasi publik terlihat di browser dan tidak boleh mengandung rahasia.

Situs web Crateview menggunakan kembali tanda UI registri, default tema, dan nama variabel merek. Situs web dikelola secara terpisah; konfigurasi sumber dan penerapannya bersifat pribadi. Sumber dokumentasinya bersifat publik di [crateview/docs](https://github.com/crateview/docs).
