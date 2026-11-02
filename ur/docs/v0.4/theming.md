---
title: "برانڈنگ اور تھیمنگ"
description: "Nuxt UI کے مقامی ڈیزائن سسٹم کو برقرار رکھتے ہوئے اپنی شناخت کا استعمال کریں۔"
---

![برانڈنگ اور تھیمنگ](https://crateview.de/images/docs/v0.4/settings-light.webp)

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

Crateview ویب سائٹ رجسٹری UI کے نشان، تھیم ڈیفالٹس اور برانڈنگ متغیر ناموں کو دوبارہ استعمال کرتی ہے۔ ویب سائٹ کو الگ سے رکھا جاتا ہے۔ اس کا ماخذ اور تعیناتی کی ترتیب نجی ہے۔ اس کے دستاویزات کے ذرائع [crateview/docs](https://github.com/crateview/docs) میں عوامی ہیں۔
