---
title: "ब्रांडिंग और थीम"
description: "Nuxt UI के मूल डिज़ाइन सिस्टम को बनाए रखते हुए अपनी स्वयं की पहचान का उपयोग करें।"
---

## चूक

Crateview Nuxt UI के `red` प्राथमिक पैलेट और `zinc` न्यूट्रल पैलेट का उपयोग करता है। डिफ़ॉल्ट रंग मोड सिस्टम है; उपयोगकर्ता आपके रंग कॉन्फ़िगरेशन से स्वतंत्र रूप से हल्का या गहरा चुन सकते हैं।

## रनटाइम थीम वैरिएबल

```dotenv
NUXT_PUBLIC_THEME_PRIMARY=red
NUXT_PUBLIC_THEME_NEUTRAL=zinc
NUXT_PUBLIC_THEME_RADIUS=0.5rem
NUXT_PUBLIC_THEME_FONT_FAMILY=Inter, ui-sans-serif, system-ui, sans-serif
```

प्राथमिक पैलेट मानक टेलविंड उच्चारण रंगों जैसे `blue`, `emerald` या `violet` को स्वीकार करता है। न्यूट्रल `slate`, `gray`, `zinc`, `neutral` या `stone` स्वीकार करता है। रेडियस गैर-ऋणात्मक `px` या `rem` मान स्वीकार करता है। फ़ॉन्ट कॉन्फ़िगरेशन एक फ़ॉन्ट स्टैक का चयन करता है; फ़ॉन्ट फ़ाइलों को बंडल करें और यदि आपका फ़ॉन्ट क्लाइंट पर स्थापित नहीं है तो `@font-face` घोषित करें।

उत्पादन परिवेश चर बदलने के बाद सर्वर को पुनरारंभ करें। इन रनटाइम सेटिंग्स के लिए किसी पुनर्निर्माण की आवश्यकता नहीं है।

## अनुप्रयोग और घटक विन्यास

डिफ़ॉल्ट को `app/app.config.ts` में परिभाषित किया गया है:

```ts
export default defineAppConfig({
  ui: { colors: { primary: 'red', neutral: 'zinc' } },
  theme: {
    radius: '0.5rem',
    fontFamily: 'Inter, ui-sans-serif, system-ui, sans-serif',
  },
})
```

घटक वेरिएंट और आइकन के लिए सामान्य Nuxt UI `ui` कॉन्फ़िगरेशन का उपयोग करें। अतिरिक्त सीएसएस डिज़ाइन टोकन और प्रकाश/अंधेरे सतह नियम `app/assets/css/main.css` में रहते हैं। स्रोत परिवर्तन के लिए उत्पादन अनुप्रयोग के पुनर्निर्माण की आवश्यकता होती है।

## ब्रांड बदलें

```dotenv
NUXT_PUBLIC_BRAND_NAME=My Registry
NUXT_PUBLIC_BRAND_LOGO_LIGHT=/my-logo.png
NUXT_PUBLIC_BRAND_LOGO_DARK=/my-logo.png
NUXT_PUBLIC_BRAND_FAVICON=/my-favicon.ico
NUXT_PUBLIC_BRAND_TITLE_TEMPLATE=%s · My Registry
```

निर्माण से पहले स्थानीय संपत्तियों को `public/` में रखें, या HTTP(S) संपत्ति यूआरएल का उपयोग करें। मौजूदा वैकल्पिक उत्पाद, दस्तावेज़ीकरण और रिपॉजिटरी लिंक को उनके संबंधित `NUXT_PUBLIC_BRAND_*_URL` चर के माध्यम से कॉन्फ़िगर किया जा सकता है। सार्वजनिक कॉन्फ़िगरेशन ब्राउज़र में दिखाई देता है और इसमें कभी भी रहस्य नहीं होना चाहिए।

Crateview वेबसाइट रजिस्ट्री यूआई के चिह्न, थीम डिफ़ॉल्ट और ब्रांडिंग चर नामों का पुन: उपयोग करती है। वेबसाइट का रखरखाव अलग से किया जाता है; इसका स्रोत और परिनियोजन कॉन्फ़िगरेशन निजी है. इसके दस्तावेज़ीकरण स्रोत [crateview/docs](https://github.com/crateview/docs) में सार्वजनिक हैं।
