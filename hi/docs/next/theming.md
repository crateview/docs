---
title: 'ब्रांडिंग और थीम'
description: 'Nuxt UI के मूल डिज़ाइन सिस्टम को बनाए रखते हुए अपनी स्वयं की पहचान का उपयोग करें।'
---

![ब्रांडिंग और थीम](https://crateview.de/images/docs/next/settings-light.webp)

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

### दोबारा बिल्ड किए बिना सर्वर फ़ाइलें

आप सर्वर की निजी फ़ाइलों के पथ भी कॉन्फ़िगर कर सकते हैं। हर कॉन्फ़िगर की गई फ़ाइल अपने संबंधित सार्वजनिक URL से प्राथमिकता लेती है; फ़ाइल वैरिएबल खाली होने पर URL सेटिंग बनी रहती है। पूर्ण पथ समर्थित हैं; सापेक्ष पथ सर्वर की कार्यशील डायरेक्टरी (Docker इमेज में `/app`) से हल होते हैं। फ़ाइल पथ कभी ब्राउज़र को नहीं भेजे जाते। एसेट स्वयं सार्वजनिक हैं, साइन इन से पहले भी।

```dotenv
NUXT_BRAND_LOGO_LIGHT_FILE=/app/branding/logo-light.svg
NUXT_BRAND_LOGO_DARK_FILE=/app/branding/logo-dark.svg
NUXT_BRAND_FAVICON_FILE=/app/branding/favicon.ico
```

समर्थित एक्सटेंशन: SVG, PNG, JPG/JPEG, WebP, GIF और ICO। फ़ाइलें पढ़ने योग्य, सामान्य और गैर-खाली होनी चाहिए; अमान्य सेटिंग सर्वर को शुरू होने से रोकती है। फ़ाइलें शुरू होते समय लोड होती हैं: पथ या सामग्री बदलने के बाद पुनः शुरू करें। दोबारा बिल्ड करना ज़रूरी नहीं है।

Docker Compose के लिए `compose.branding.yaml` ओवरराइड फ़ाइल बनाएँ:

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

होस्ट के `./branding` में तीनों फ़ाइलें बनाएँ और कंटेनर के `node` उपयोगकर्ता को उन्हें पढ़ने दें। `docker compose -f compose.yaml -f compose.branding.yaml up -d` चलाएँ। माउंट केवल पढ़ने के लिए है; कॉन्फ़िगर किए गए पथ कंटेनर के अंदर की फ़ाइलों के हैं। ये निजी फ़ाइल वैरिएबल रजिस्ट्री UI पर लागू होते हैं।

Crateview वेबसाइट रजिस्ट्री यूआई के चिह्न, थीम डिफ़ॉल्ट और ब्रांडिंग चर नामों का पुन: उपयोग करती है। वेबसाइट का रखरखाव अलग से किया जाता है; इसका स्रोत और परिनियोजन कॉन्फ़िगरेशन निजी है. इसके दस्तावेज़ीकरण स्रोत [crateview/docs](https://github.com/crateview/docs) में सार्वजनिक हैं।

## इंस्टॉल किए गए ऐप की ब्रांडिंग

ऐप `NUXT_PUBLIC_BRAND_NAME` और `NUXT_PUBLIC_BRAND_DESCRIPTION` उपयोग करता है। PWA आइकन हल्के/गहरे लोगो और favicon से स्वतंत्र हैं। 192 पिक्सेल PNG, Apple टच आइकन भी है। डिफ़ॉल्ट आइकन Crateview चिह्न उपयोग करते हैं।

होस्ट पर बिल्कुल 192×192 और 512×512 पिक्सेल की दोनों PNG बनाएँ और कंटेनर उपयोगकर्ता को पढ़ने दें। नीचे का ओवरले उन्हें केवल पढ़ने के लिए माउंट करता है; मौजूदा Compose फ़ाइल के साथ उपयोग करें। URLs के लिए `NUXT_PUBLIC_BRAND_PWA_ICON_192` और `NUXT_PUBLIC_BRAND_PWA_ICON_512` उपयोग करें।

```yaml
services:
  frontend:
    volumes:
      - ./branding:/app/branding:ro
    environment:
      NUXT_BRAND_PWA_ICON_192_FILE: /app/branding/pwa-192.png
      NUXT_BRAND_PWA_ICON_512_FILE: /app/branding/pwa-512.png
```

ब्राउज़र इंस्टॉल किए गए ऐप का मेटाडेटा सँभालता है, इसलिए आइकन बदलाव देर से दिख सकते हैं। Crateview पुनः शुरू करने से कॉन्फ़िगरेशन लागू होता है, लेकिन मौजूदा आइकन तुरंत बदलने की गारंटी नहीं है।
