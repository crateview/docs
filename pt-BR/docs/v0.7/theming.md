---
title: 'Marca e tema'
description: 'Use sua própria identidade enquanto mantém o sistema de design nativo da UI Nuxt.'
---

![Marca e tema](https://crateview.de/images/docs/v0.7/settings-light.webp)

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

### Arquivos do servidor sem recompilar

Você também pode configurar caminhos privados de arquivos no servidor. Cada arquivo configurado tem prioridade sobre sua URL pública correspondente; uma variável de arquivo vazia mantém a URL. Caminhos absolutos são aceitos; os relativos partem do diretório de trabalho do servidor (`/app` na imagem Docker). Os caminhos nunca são enviados ao navegador. Os recursos são públicos, inclusive antes do login.

```dotenv
NUXT_BRAND_LOGO_LIGHT_FILE=/app/branding/logo-light.svg
NUXT_BRAND_LOGO_DARK_FILE=/app/branding/logo-dark.svg
NUXT_BRAND_FAVICON_FILE=/app/branding/favicon.ico
```

Extensões aceitas: SVG, PNG, JPG/JPEG, WebP, GIF e ICO. Os arquivos devem ser regulares, legíveis e não vazios; configurações inválidas impedem a inicialização do servidor. Os arquivos são carregados na inicialização: reinicie após alterar caminhos ou conteúdos. Não é necessário recompilar.

Para Docker Compose, crie um arquivo de substituição `compose.branding.yaml`:

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

Crie os três arquivos em `./branding` no host e permita a leitura pelo usuário `node` do contêiner. Execute `docker compose -f compose.yaml -f compose.branding.yaml up -d`. A montagem é somente leitura; os caminhos configurados apontam para arquivos dentro do contêiner. Essas variáveis privadas de arquivos se aplicam à interface do registro.

O site Crateview reutiliza a marca da interface do usuário do registro, padrões de tema e nomes de variáveis ​​de marca. O site é mantido separadamente; sua configuração de origem e implantação é privada. Suas fontes de documentação são públicas em [crateview/docs](https://github.com/crateview/docs).

## Identidade do aplicativo instalado

O aplicativo usa `NUXT_PUBLIC_BRAND_NAME` e `NUXT_PUBLIC_BRAND_DESCRIPTION`. Os ícones PWA são independentes dos logotipos claro/escuro e do favicon. O PNG de 192 pixels também fornece o ícone de toque da Apple. O padrão usa a marca Crateview.

Crie no host os dois PNGs de exatamente 192×192 e 512×512 pixels, legíveis pelo usuário do contêiner. A sobreposição abaixo monta os arquivos somente para leitura; combine-a ao Compose existente. Para URLs, use `NUXT_PUBLIC_BRAND_PWA_ICON_192` e `NUXT_PUBLIC_BRAND_PWA_ICON_512`.

```yaml
services:
  frontend:
    volumes:
      - ./branding:/app/branding:ro
    environment:
      NUXT_BRAND_PWA_ICON_192_FILE: /app/branding/pwa-192.png
      NUXT_BRAND_PWA_ICON_512_FILE: /app/branding/pwa-512.png
```

Os navegadores gerenciam os metadados dos aplicativos instalados; mudanças de ícone podem demorar. Reiniciar o Crateview aplica a configuração, mas não garante atualização imediata de ícones já instalados.
