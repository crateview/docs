---
title: "Marca e tema"
description: "Use sua própria identidade enquanto mantém o sistema de design nativo da UI Nuxt."
---

![Marca e tema](https://crateview.de/images/docs/v0.5/settings-light.webp)

## Padrões

Crateview usa a paleta primária `red` da UI Nuxt e a paleta neutra `zinc`. O modo de cor padrão é Sistema; os usuários podem escolher Claro ou Escuro independentemente da configuração de cores.

## Variáveis ​​de tema de tempo de execução

```dotenv
NUXT_PUBLIC_THEME_PRIMARY=red
NUXT_PUBLIC_THEME_NEUTRAL=zinc
NUXT_PUBLIC_THEME_RADIUS=0.5rem
NUXT_PUBLIC_THEME_FONT_FAMILY=Inter, ui-sans-serif, system-ui, sans-serif
```

A paleta primária aceita cores de destaque padrão do Tailwind, como `blue`, `emerald` ou `violet`. Neutro aceita `slate`, `gray`, `zinc`, `neutral` ou `stone`. O raio aceita valores `px` ou `rem` não negativos. A configuração de fontes seleciona uma pilha de fontes; agrupe arquivos de fonte e declare `@font-face` se sua fonte não estiver instalada no cliente.

Reinicie o servidor após alterar as variáveis ​​do ambiente de produção. Nenhuma reconstrução é necessária para essas configurações de tempo de execução.

## Configuração de aplicativos e componentes

Os padrões são definidos em `app/app.config.ts`:

```ts
export default defineAppConfig({
  ui: { colors: { primary: 'red', neutral: 'zinc' } },
  theme: {
    radius: '0.5rem',
    fontFamily: 'Inter, ui-sans-serif, system-ui, sans-serif',
  },
})
```

Use a configuração normal da UI Nuxt `ui` para variantes de componentes e ícones. Tokens de design CSS adicionais e regras de superfície clara/escura residem em `app/assets/css/main.css`. As alterações de origem exigem a reconstrução do aplicativo de produção.

## Substitua a marca

```dotenv
NUXT_PUBLIC_BRAND_NAME=My Registry
NUXT_PUBLIC_BRAND_LOGO_LIGHT=/my-logo.png
NUXT_PUBLIC_BRAND_LOGO_DARK=/my-logo.png
NUXT_PUBLIC_BRAND_FAVICON=/my-favicon.ico
NUXT_PUBLIC_BRAND_TITLE_TEMPLATE=%s · My Registry
```

Coloque ativos locais em `public/` antes de compilar ou use URLs de ativos HTTP(S). Links opcionais existentes de produtos, documentação e repositórios podem ser configurados por meio de suas variáveis ​​`NUXT_PUBLIC_BRAND_*_URL` correspondentes. A configuração pública é visível no navegador e nunca deve conter segredos.

O site Crateview reutiliza a marca da interface do usuário do registro, padrões de tema e nomes de variáveis ​​de marca. O site é mantido separadamente; sua configuração de origem e implantação é privada. Suas fontes de documentação são públicas em [crateview/docs](https://github.com/crateview/docs).
