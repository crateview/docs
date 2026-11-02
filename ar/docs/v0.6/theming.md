---
title: 'العلامات التجارية والموضوعات'
description: 'استخدم هويتك الخاصة مع الحفاظ على نظام التصميم الأصلي لـ Nuxt UI.'
---

![العلامات التجارية والموضوعات](https://crateview.de/images/docs/v0.6/settings-light.webp)

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

### ملفات الخادم دون إعادة البناء

يمكنك أيضًا ضبط مسارات ملفات خاصة على الخادم. يتقدم كل ملف مضبوط على عنوان URL العام المقابل له؛ وإبقاء متغير الملف فارغًا يحافظ على إعداد URL. تُدعم المسارات المطلقة؛ وتُحل المسارات النسبية انطلاقًا من مجلد عمل الخادم (`/app` في صورة Docker). لا تُرسل مسارات الملفات إلى المتصفح. الأصول نفسها عامة، حتى قبل تسجيل الدخول.

```dotenv
NUXT_BRAND_LOGO_LIGHT_FILE=/app/branding/logo-light.svg
NUXT_BRAND_LOGO_DARK_FILE=/app/branding/logo-dark.svg
NUXT_BRAND_FAVICON_FILE=/app/branding/favicon.ico
```

الامتدادات المدعومة: SVG وPNG وJPG/JPEG وWebP وGIF وICO. يجب أن تكون الملفات عادية وقابلة للقراءة وغير فارغة؛ وتمنع الإعدادات غير الصالحة بدء الخادم. تُحمّل الملفات عند البدء: أعد التشغيل بعد تغيير المسارات أو محتويات الملفات. لا حاجة لإعادة البناء.

لاستخدام Docker Compose، أنشئ ملف التجاوز `compose.branding.yaml`:

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

أنشئ الملفات الثلاثة في `./branding` على المضيف واسمح لمستخدم الحاوية `node` بقراءتها. شغّل `docker compose -f compose.yaml -f compose.branding.yaml up -d`. الربط للقراءة فقط؛ وتشير المسارات المضبوطة إلى ملفات داخل الحاوية. تنطبق متغيرات الملفات الخاصة هذه على واجهة السجل.

يعيد موقع الويب Crateview استخدام علامة واجهة مستخدم التسجيل والإعدادات الافتراضية للموضوع وأسماء متغيرات العلامة التجارية. تتم صيانة الموقع بشكل منفصل؛ مصدره وتكوين النشر خاصان. مصادر التوثيق الخاصة به متاحة للعامة في [crateview/docs](https://github.com/crateview/docs).

## هوية التطبيق المثبّت

يستخدم التطبيق `NUXT_PUBLIC_BRAND_NAME` و`NUXT_PUBLIC_BRAND_DESCRIPTION`. أيقونات PWA مستقلة عن شعارات الوضع الفاتح والداكن وfavicon. تُستخدم صورة PNG بحجم 192 بكسل أيضًا كأيقونة لمس Apple. الأيقونات الافتراضية تحمل علامة Crateview.

أنشئ على المضيف ملفي PNG بأبعاد 192×192 و512×512 بكسل تمامًا، واسمح لمستخدم الحاوية بقراءتهما. يربط التكوين الإضافي التالي الملفات للقراءة فقط؛ ادمجه مع ملف Compose الحالي. للعناوين، استخدم `NUXT_PUBLIC_BRAND_PWA_ICON_192` و`NUXT_PUBLIC_BRAND_PWA_ICON_512`.

```yaml
services:
  frontend:
    volumes:
      - ./branding:/app/branding:ro
    environment:
      NUXT_BRAND_PWA_ICON_192_FILE: /app/branding/pwa-192.png
      NUXT_BRAND_PWA_ICON_512_FILE: /app/branding/pwa-512.png
```

قد تتأخر تغييرات الأيقونات لأن المتصفح يدير بيانات التطبيقات المثبتة. تطبق إعادة تشغيل Crateview الإعدادات، لكنها لا تضمن تحديث أيقونة التثبيت الحالية فورًا.
