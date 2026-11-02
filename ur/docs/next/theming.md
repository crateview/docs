---
title: 'برانڈنگ اور تھیمنگ'
description: 'Nuxt UI کے مقامی ڈیزائن سسٹم کو برقرار رکھتے ہوئے اپنی شناخت کا استعمال کریں۔'
---

![برانڈنگ اور تھیمنگ](https://crateview.de/images/docs/next/settings-light.webp)

## ڈیفالٹس

Crateview Nuxt UI کا `red` بنیادی پیلیٹ اور `zinc` غیر جانبدار پیلیٹ استعمال کرتا ہے۔ ڈیفالٹ کلر موڈ سسٹم ہے۔ صارفین آپ کے رنگ کی ترتیب سے آزادانہ طور پر ہلکے یا گہرے کا انتخاب کر سکتے ہیں۔

## رن ٹائم تھیم متغیرات

```dotenv
NUXT_PUBLIC_THEME_PRIMARY=red
NUXT_PUBLIC_THEME_NEUTRAL=zinc
NUXT_PUBLIC_THEME_RADIUS=0.5rem
NUXT_PUBLIC_THEME_FONT_FAMILY=Inter, ui-sans-serif, system-ui, sans-serif
```

بنیادی پیلیٹ معیاری ٹیل ونڈ لہجے کے رنگوں جیسے `blue`، `emerald` یا `violet` کو قبول کرتا ہے۔ غیر جانبدار `slate`، `gray`، `zinc`، `neutral` یا `stone` کو قبول کرتا ہے۔ رداس غیر منفی `px` یا `rem` اقدار کو قبول کرتا ہے۔ فونٹ کنفیگریشن فونٹ اسٹیک کو منتخب کرتی ہے۔ فونٹ فائلوں کو بنڈل کریں اور `@font-face` کا اعلان کریں اگر آپ کا فونٹ کلائنٹ پر انسٹال نہیں ہے۔

پیداوار کے ماحول کے متغیرات کو تبدیل کرنے کے بعد سرور کو دوبارہ شروع کریں۔ ان رن ٹائم ترتیبات کے لیے دوبارہ تعمیر کی ضرورت نہیں ہے۔

## درخواست اور اجزاء کی ترتیب

ڈیفالٹس کی تعریف `app/app.config.ts` میں کی گئی ہے:

```ts
export default defineAppConfig({
  ui: { colors: { primary: 'red', neutral: 'zinc' } },
  theme: {
    radius: '0.5rem',
    fontFamily: 'Inter, ui-sans-serif, system-ui, sans-serif',
  },
})
```

اجزاء کی مختلف حالتوں اور آئیکنز کے لیے عام Nuxt UI `ui` کنفیگریشن استعمال کریں۔ اضافی CSS ڈیزائن ٹوکن اور روشنی/گہرے سطح کے اصول `app/assets/css/main.css` میں رہتے ہیں۔ ماخذ کی تبدیلیوں کے لیے پروڈکشن ایپلیکیشن کی تعمیر نو کی ضرورت ہوتی ہے۔

## برانڈ کو تبدیل کریں۔

```dotenv
NUXT_PUBLIC_BRAND_NAME=My Registry
NUXT_PUBLIC_BRAND_LOGO_LIGHT=/my-logo.png
NUXT_PUBLIC_BRAND_LOGO_DARK=/my-logo.png
NUXT_PUBLIC_BRAND_FAVICON=/my-favicon.ico
NUXT_PUBLIC_BRAND_TITLE_TEMPLATE=%s · My Registry
```

تعمیر کرنے سے پہلے مقامی اثاثوں کو `public/` میں رکھیں، یا HTTP(S) اثاثہ URLs کا استعمال کریں۔ موجودہ اختیاری پروڈکٹ، دستاویزات اور ریپوزٹری لنکس کو ان کے متعلقہ `NUXT_PUBLIC_BRAND_*_URL` متغیرات کے ذریعے ترتیب دیا جا سکتا ہے۔ عوامی ترتیب براؤزر میں نظر آتی ہے اور اس میں کبھی بھی راز نہیں ہونا چاہیے۔

### دوبارہ بلڈ کیے بغیر سرور کی فائلیں

آپ سرور پر نجی فائلوں کے راستے بھی ترتیب دے سکتے ہیں۔ ہر ترتیب دی گئی فائل کو متعلقہ عوامی URL پر ترجیح حاصل ہے؛ فائل کا متغیر خالی ہو تو URL کی ترتیب برقرار رہتی ہے۔ مطلق راستے معاون ہیں؛ نسبتی راستے سرور کی ورکنگ ڈائریکٹری (Docker امیج میں `/app`) سے طے ہوتے ہیں۔ فائلوں کے راستے کبھی براؤزر کو نہیں بھیجے جاتے۔ اثاثے خود عوامی ہیں، سائن ان سے پہلے بھی۔

```dotenv
NUXT_BRAND_LOGO_LIGHT_FILE=/app/branding/logo-light.svg
NUXT_BRAND_LOGO_DARK_FILE=/app/branding/logo-dark.svg
NUXT_BRAND_FAVICON_FILE=/app/branding/favicon.ico
```

معاون ایکسٹینشن: SVG، PNG، JPG/JPEG، WebP، GIF اور ICO۔ فائلیں عام، قابل مطالعہ اور غیر خالی ہونی چاہئیں؛ غلط ترتیبات سرور کو شروع ہونے سے روکتی ہیں۔ فائلیں آغاز پر لوڈ ہوتی ہیں: راستے یا مواد بدلنے کے بعد دوبارہ شروع کریں۔ دوبارہ بلڈ کی ضرورت نہیں۔

Docker Compose کے لیے `compose.branding.yaml` اووررائیڈ فائل بنائیں:

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

ہوسٹ پر `./branding` میں تینوں فائلیں بنائیں اور کنٹینر کے `node` صارف کو انہیں پڑھنے کی اجازت دیں۔ `docker compose -f compose.yaml -f compose.branding.yaml up -d` چلائیں۔ ماؤنٹ صرف پڑھنے کے لیے ہے؛ ترتیب دیے گئے راستے کنٹینر کے اندر کی فائلوں کے ہیں۔ یہ نجی فائل متغیرات رجسٹری UI پر لاگو ہوتے ہیں۔

Crateview ویب سائٹ رجسٹری UI کے نشان، تھیم ڈیفالٹس اور برانڈنگ متغیر ناموں کو دوبارہ استعمال کرتی ہے۔ ویب سائٹ کو الگ سے رکھا جاتا ہے۔ اس کا ماخذ اور تعیناتی کی ترتیب نجی ہے۔ اس کے دستاویزات کے ذرائع [crateview/docs](https://github.com/crateview/docs) میں عوامی ہیں۔

## انسٹال شدہ ایپ کی برانڈنگ

ایپ `NUXT_PUBLIC_BRAND_NAME` اور `NUXT_PUBLIC_BRAND_DESCRIPTION` استعمال کرتی ہے۔ PWA آئیکن روشن/تاریک لوگو اور favicon سے الگ ہیں۔ 192 پکسل PNG، Apple ٹچ آئیکن بھی فراہم کرتا ہے۔ طے شدہ آئیکن Crateview کا نشان استعمال کرتے ہیں۔

ہوسٹ پر عین 192×192 اور 512×512 پکسل کی دو PNG فائلیں بنائیں اور کنٹینر صارف کو پڑھنے کی اجازت دیں۔ درج ذیل اوورلے انہیں صرف پڑھنے کے لیے ماؤنٹ کرتا ہے؛ موجودہ Compose فائل کے ساتھ ملائیں۔ URL کے لیے `NUXT_PUBLIC_BRAND_PWA_ICON_192` اور `NUXT_PUBLIC_BRAND_PWA_ICON_512` استعمال کریں۔

```yaml
services:
  frontend:
    volumes:
      - ./branding:/app/branding:ro
    environment:
      NUXT_BRAND_PWA_ICON_192_FILE: /app/branding/pwa-192.png
      NUXT_BRAND_PWA_ICON_512_FILE: /app/branding/pwa-512.png
```

براؤزر انسٹال شدہ ایپ کا میٹاڈیٹا سنبھالتا ہے، اس لیے آئیکن کی تبدیلی دیر سے دکھ سکتی ہے۔ Crateview دوبارہ شروع کرنے سے ترتیب نافذ ہوتی ہے، مگر پہلے سے انسٹال آئیکن کا فوری ریفریش یقینی نہیں۔
