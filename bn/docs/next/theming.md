---
title: 'ব্র্যান্ডিং এবং থিমিং'
description: 'Nuxt UI এর নেটিভ ডিজাইন সিস্টেম রাখার সময় আপনার নিজস্ব পরিচয় ব্যবহার করুন।'
---

![ব্র্যান্ডিং এবং থিমিং](https://crateview.de/images/docs/next/settings-light.webp)

## ডিফল্ট

Crateview Nuxt UI এর `red` প্রাথমিক প্যালেট এবং `zinc` নিরপেক্ষ প্যালেট ব্যবহার করে। ডিফল্ট রঙ মোড হল সিস্টেম; ব্যবহারকারীরা আপনার রঙের কনফিগারেশন থেকে স্বাধীনভাবে হালকা বা অন্ধকার চয়ন করতে পারেন।

## রানটাইম থিম ভেরিয়েবল

```dotenv
NUXT_PUBLIC_THEME_PRIMARY=red
NUXT_PUBLIC_THEME_NEUTRAL=zinc
NUXT_PUBLIC_THEME_RADIUS=0.5rem
NUXT_PUBLIC_THEME_FONT_FAMILY=Inter, ui-sans-serif, system-ui, sans-serif
```

প্রাথমিক প্যালেট মানসম্মত Tailwind অ্যাকসেন্ট রং যেমন `blue`, `emerald` বা `violet` গ্রহণ করে। নিরপেক্ষ `slate`, `gray`, `zinc`, `neutral` বা `stone` গ্রহণ করে। ব্যাসার্ধ নেতিবাচক `px` বা `rem` মান গ্রহণ করে। ফন্ট কনফিগারেশন একটি ফন্ট স্ট্যাক নির্বাচন করে; ফন্ট ফাইল বান্ডিল করুন এবং আপনার ফন্ট ক্লায়েন্টে ইনস্টল না থাকলে `@font-face` ঘোষণা করুন।

উত্পাদন পরিবেশ ভেরিয়েবল পরিবর্তন করার পরে সার্ভার পুনরায় আরম্ভ করুন. এই রানটাইম সেটিংসের জন্য কোন পুনর্নির্মাণের প্রয়োজন নেই।

## অ্যাপ্লিকেশন এবং উপাদান কনফিগারেশন

ডিফল্টগুলি `app/app.config.ts` এ সংজ্ঞায়িত করা হয়েছে:

```ts
export default defineAppConfig({
  ui: { colors: { primary: 'red', neutral: 'zinc' } },
  theme: {
    radius: '0.5rem',
    fontFamily: 'Inter, ui-sans-serif, system-ui, sans-serif',
  },
})
```

কম্পোনেন্ট ভেরিয়েন্ট এবং আইকনগুলির জন্য স্বাভাবিক Nuxt UI `ui` কনফিগারেশন ব্যবহার করুন। অতিরিক্ত CSS ডিজাইন টোকেন এবং হালকা/অন্ধকার পৃষ্ঠের নিয়মগুলি `app/assets/css/main.css`-এ থাকে। উত্স পরিবর্তনের জন্য উত্পাদন অ্যাপ্লিকেশন পুনর্নির্মাণের প্রয়োজন৷

## ব্র্যান্ড প্রতিস্থাপন

```dotenv
NUXT_PUBLIC_BRAND_NAME=My Registry
NUXT_PUBLIC_BRAND_LOGO_LIGHT=/my-logo.png
NUXT_PUBLIC_BRAND_LOGO_DARK=/my-logo.png
NUXT_PUBLIC_BRAND_FAVICON=/my-favicon.ico
NUXT_PUBLIC_BRAND_TITLE_TEMPLATE=%s · My Registry
```

নির্মাণের আগে স্থানীয় সম্পদগুলিকে `public/`-এ রাখুন বা HTTP(S) সম্পদ URL ব্যবহার করুন৷ বিদ্যমান ঐচ্ছিক পণ্য, ডকুমেন্টেশন এবং সংগ্রহস্থলের লিঙ্কগুলি তাদের সংশ্লিষ্ট `NUXT_PUBLIC_BRAND_*_URL` ভেরিয়েবলের মাধ্যমে কনফিগার করা যেতে পারে। সর্বজনীন কনফিগারেশন ব্রাউজারে দৃশ্যমান এবং কখনই গোপনীয়তা থাকা উচিত নয়।

### পুনরায় বিল্ড ছাড়াই সার্ভারের ফাইল

সার্ভারের ব্যক্তিগত ফাইলের পথও কনফিগার করতে পারেন। প্রতিটি কনফিগার করা ফাইল সংশ্লিষ্ট পাবলিক URL-এর চেয়ে অগ্রাধিকার পায়; ফাইল ভেরিয়েবল খালি থাকলে URL সেটিং বজায় থাকে। অ্যাবসোলিউট পাথ সমর্থিত; রিলেটিভ পাথ সার্ভারের কার্যকরী ডিরেক্টরি (Docker ইমেজে `/app`) থেকে নির্ধারিত হয়। ফাইলের পথ কখনো ব্রাউজারে পাঠানো হয় না। অ্যাসেটগুলো নিজে সর্বজনীন, সাইন ইন করার আগেও।

```dotenv
NUXT_BRAND_LOGO_LIGHT_FILE=/app/branding/logo-light.svg
NUXT_BRAND_LOGO_DARK_FILE=/app/branding/logo-dark.svg
NUXT_BRAND_FAVICON_FILE=/app/branding/favicon.ico
```

সমর্থিত এক্সটেনশন: SVG, PNG, JPG/JPEG, WebP, GIF ও ICO। ফাইলগুলো পাঠযোগ্য, সাধারণ ও অখালি হতে হবে; অবৈধ সেটিং সার্ভার চালু হতে দেয় না। চালুর সময় ফাইল লোড হয়: পথ বা ফাইলের বিষয়বস্তু বদলালে পুনরায় চালু করুন। আবার বিল্ড করার প্রয়োজন নেই।

Docker Compose-এর জন্য `compose.branding.yaml` ওভাররাইড ফাইল তৈরি করুন:

```yaml
services:
  frontend:
    volumes:
      - ./branding:/app/branding:ro
    environment:
      NUXT_BRAND_LOGO_LIGHT_FILE: /app/branding/logo-light.svg
      NUXT_BRAND_LOGO_DARK_FILE: /app/branding/logo-dark.svg
      NUXT_BRAND_FAVICON_FILE: /app/branding/favicon.ico
```

হোস্টের `./branding`-এ তিনটি ফাইল তৈরি করুন এবং কন্টেইনারের `node` ব্যবহারকারীকে পড়ার অনুমতি দিন। `docker compose -f compose.yaml -f compose.branding.yaml up -d` চালান। মাউন্ট শুধু পড়ার জন্য; কনফিগার করা পথ কন্টেইনারের ভেতরের ফাইল নির্দেশ করে। এই ব্যক্তিগত ফাইল ভেরিয়েবলগুলো রেজিস্ট্রি UI-এর জন্য প্রযোজ্য।

Crateview ওয়েবসাইট রেজিস্ট্রি UI এর চিহ্ন, থিম ডিফল্ট এবং ব্র্যান্ডিং পরিবর্তনশীল নামগুলি পুনরায় ব্যবহার করে৷ ওয়েবসাইট আলাদাভাবে রক্ষণাবেক্ষণ করা হয়; এর উৎস এবং স্থাপনার কনফিগারেশন ব্যক্তিগত। এর ডকুমেন্টেশন উত্সগুলি [crateview/docs](https://github.com/crateview/docs)-এ সর্বজনীন।
