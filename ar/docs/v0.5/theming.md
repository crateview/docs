---
title: "العلامات التجارية والموضوعات"
description: "استخدم هويتك الخاصة مع الحفاظ على نظام التصميم الأصلي لـ Nuxt UI."
---

![العلامات التجارية والموضوعات](https://crateview.de/images/docs/v0.5/settings-light.webp)

## الافتراضيات

يستخدم Crateview لوحة الألوان الأساسية `red` الخاصة بـ Nuxt ولوحة الألوان المحايدة `zinc`. وضع اللون الافتراضي هو النظام؛ يمكن للمستخدمين اختيار فاتح أو داكن بشكل مستقل عن تكوين الألوان الخاص بك.

## متغيرات موضوع وقت التشغيل

```dotenv
NUXT_PUBLIC_THEME_PRIMARY=red
NUXT_PUBLIC_THEME_NEUTRAL=zinc
NUXT_PUBLIC_THEME_RADIUS=0.5rem
NUXT_PUBLIC_THEME_FONT_FAMILY=Inter, ui-sans-serif, system-ui, sans-serif
```

تقبل اللوحة الأساسية الألوان المميزة للرياح الخلفية مثل `blue` أو `emerald` أو `violet`. يقبل الحياد `slate` أو `gray` أو `zinc` أو `neutral` أو `stone`. يقبل Radius قيم `px` أو `rem` غير السالبة. تكوين الخط يحدد مكدس الخطوط؛ قم بتجميع ملفات الخطوط وأعلن عن `@font-face` إذا لم يتم تثبيت الخط الخاص بك على العميل.

أعد تشغيل الخادم بعد تغيير متغيرات بيئة الإنتاج. ليست هناك حاجة إلى إعادة بناء لإعدادات وقت التشغيل هذه.

## تكوين التطبيق والمكونات

تم تعريف الإعدادات الافتراضية في `app/app.config.ts`:

```ts
export default defineAppConfig({
  ui: { colors: { primary: 'red', neutral: 'zinc' } },
  theme: {
    radius: '0.5rem',
    fontFamily: 'Inter, ui-sans-serif, system-ui, sans-serif',
  },
})
```

استخدم تكوين Nuxt UI العادي `ui` لمتغيرات المكونات والأيقونات. توجد رموز تصميم CSS الإضافية وقواعد السطح الفاتح/المظلم في `app/assets/css/main.css`. تتطلب تغييرات المصدر إعادة إنشاء تطبيق الإنتاج.

## استبدل العلامة التجارية

```dotenv
NUXT_PUBLIC_BRAND_NAME=My Registry
NUXT_PUBLIC_BRAND_LOGO_LIGHT=/my-logo.png
NUXT_PUBLIC_BRAND_LOGO_DARK=/my-logo.png
NUXT_PUBLIC_BRAND_FAVICON=/my-favicon.ico
NUXT_PUBLIC_BRAND_TITLE_TEMPLATE=%s · My Registry
```

ضع الأصول المحلية في `public/` قبل إنشاء عناوين URL لأصول HTTP(S) أو استخدمها. يمكن تكوين روابط المنتج والوثائق والمستودع الاختيارية الحالية من خلال متغيرات `NUXT_PUBLIC_BRAND_*_URL` المقابلة لها. يكون التكوين العام مرئيًا في المتصفح ويجب ألا يحتوي أبدًا على أسرار.

يعيد موقع الويب Crateview استخدام علامة واجهة مستخدم التسجيل والإعدادات الافتراضية للموضوع وأسماء متغيرات العلامة التجارية. تتم صيانة الموقع بشكل منفصل؛ مصدره وتكوين النشر خاصان. مصادر التوثيق الخاصة به متاحة للعامة في [crateview/docs](https://github.com/crateview/docs).
