---
title: "Configuração"
description: "Conexão de registro, variáveis ​​de ambiente de tempo de execução e comportamento de implantação."
---

## Camadas de configuração

Os padrões do aplicativo são definidos em `nuxt.config.ts` e `app/app.config.ts`. As variáveis ​​de ambiente substituem os padrões de tempo de execução quando o servidor é iniciado. Os comandos de desenvolvimento e construção são `.env`; o ponto de entrada do nó de produção não o lê automaticamente.

Reinicie o servidor após alterar as variáveis ​​de ambiente de tempo de execução. A edição da configuração do componente no nível de origem requer uma nova compilação de produção.

## Formato de data e hora

Defina `NUXT_PUBLIC_DATE_FORMAT` em tempo de execução para formatar todas as datas da interface. O fuso horário vem do navegador. Se o valor estiver vazio ou for inválido, o Crateview usa a configuração regional do navegador.

Exemplo: `DD.MM.YYYY HH:mm`. Tokens: `YYYY`, `MM`, `DD`, `HH`, `mm`, `ss`, `Z` (UTC ±HH:mm). Coloque texto literal entre colchetes, por exemplo `[às]`.

## Configurações de registro e sessão

| Variável | Padrão | Propósito |
| ------------------------------ | ----------------------- | ----------------------------------- |
| `NUXT_REGISTRY_UPSTREAM` | `http://127.0.0.1:5000` | Origem do registro do lado do servidor. |
| `NUXT_REGISTRY_DELETE_ENABLED` | `false` | Exponha a exclusão do manifesto na IU. |
| `NUXT_SESSION_PASSWORD` | Aleatório por processo | Segredo usado para criptografar sessões de UI. |
| `HOST` | Dependente do tempo de execução | Interface de escuta do servidor de nó. |
| `PORT` | `3000` | Porta de escuta do servidor do nó. |

O upstream deve ser uma origem HTTP(S) acessível a partir do servidor front-end. Mantenha as credenciais fora do URL. As solicitações usam a autenticação do usuário atual; Crateview não substitui o controle de acesso ao registro.

## Variáveis ​​de marca pública

Todos os nomes abaixo usam o prefixo `NUXT_PUBLIC_BRAND_`:

| Sufixo | Propósito |
| ------------------------------------------- | ----------------------------------------------- |
| `NAME` | Nome do aplicativo. |
| `DESCRIPTION` | Descrição do aplicativo. |
| `LOGO_LIGHT`, `LOGO_DARK` | Caminhos de logotipo ou URLs HTTP(S). |
| `FAVICON` | Caminho ou URL do ícone do navegador. |
| `TITLE_TEMPLATE` | Título da página; `%s` é substituído pelo título da página. |
| `PRODUCT_URL`, `DOCS_URL`, `REPOSITORY_URL` | Links de navegação opcionais. |

A configuração pública é enviada ao navegador. Nunca armazene senhas ou tokens em variáveis ​​`NUXT_PUBLIC_*`. Consulte [marca e tema](/pt-BR/docs/v0.8/theming) para opções de cores e tipografia.

## Layout de rede

O navegador se comunica com Crateview em uma origem. Nitro faz proxy `/v2/**` para clientes de registro e lê metadados de registro por meio de `/api/registry/**`. Isso evita a necessidade de acesso de navegador de origem cruzada ao próprio registro.

Atrás de um proxy reverso, preserve a origem pública e os cabeçalhos do protocolo. O endereço de front-end público também é o endereço de registro exibido nos comandos pull copiados. Para uso autenticado remotamente, encerre o HTTPS corretamente antes de aceitar as credenciais do navegador ou da CLI.

## Compose versus configuração autônoma

O arquivo Compose incluído se conecta intencionalmente ao seu próprio serviço `registry` com `NUXT_REGISTRY_UPSTREAM=http://registry:5000`. Para usar outro registro em uma implantação de contêiner, adapte esse ambiente de serviço ou execute o front-end separadamente. Não presuma que um registro de loopback de host possa ser acessado por meio de um gateway de contêiner.
## Configuração desta versão

Monitoramento opcional, desativado por padrão. Visão geral abre o painel quando ativo; caso contrário /home redireciona para /explore. A ativação expõe dados globais a todos com acesso à interface, incluindo visitantes anônimos em registros públicos. O modo token gerenciado exige leitura global adicional.

| ENV | Default |
| --- | --- |
| `NUXT_MONITORING_ENABLED` | `false` |
| `NUXT_MONITORING_METRICS_URL` | — |
| `NUXT_MONITORING_METRICS_TOKEN` | — |
| `NUXT_MONITORING_WEBHOOK_SECRET` | — |
| `NUXT_MONITORING_DATABASE_PATH` | `.dev/monitoring.sqlite` |

[Monitoramento](/pt-BR/docs/v0.8/monitoring)

| ENV | Default |
| --- | --- |
| `NUXT_AUTH_MODE` | `off` |
| `NUXT_AUTH_SERVICE_URL` | — |
| `NUXT_AUTH_SERVICE_SECRET` | — |
| `NUXT_AUTH_TOKEN_REALM` | — |
| `NUXT_USER_CREATION_ALLOWED_USERS` | — |

Administração agrupa Configurações e Usuários. O menu da conta também oferece idioma e tema. Conta altera sua senha com confirmação da atual. A lista do servidor controla listagem e criação de contas; concessões de token são somente leitura. Novas senhas exigem no mínimo 12 caracteres e no máximo 72 bytes UTF-8. Novas contas de token começam sem acesso a repositórios.

A lista de permissões do servidor controla a listagem, criação, redefinição de senhas e exclusão de contas. O menu de três pontos alinhado à direita permite aos administradores redefinir a senha de outro usuário sem a senha antiga ou excluir sua conta após confirmação. A redefinição preserva as permissões dos repositórios; a exclusão as remove. Seu próprio menu abre Conta para alterar sua senha com a senha atual; a exclusão da própria conta é bloqueada.

## Configuração de ícones PWA

| Variável | Padrão | Finalidade |
| --- | --- | --- |
| `NUXT_PUBLIC_BRAND_PWA_ICON_192` | `/branding/pwa-192.png` | URL do ícone de instalação de 192 pixels. |
| `NUXT_PUBLIC_BRAND_PWA_ICON_512` | `/branding/pwa-512.png` | URL do ícone de instalação de 512 pixels. |
| `NUXT_BRAND_PWA_ICON_192_FILE` | — | Arquivo PNG privado de 192 pixels com prioridade. |
| `NUXT_BRAND_PWA_ICON_512_FILE` | — | Arquivo PNG privado de 512 pixels com prioridade. |

Arquivos locais prevalecem sobre as respectivas URLs públicas. Exigem PNGs legíveis, não vazios e com dimensões quadradas exatas. Arquivos inválidos impedem a inicialização. Caminhos relativos partem do diretório de trabalho do servidor; caminhos do contêiner devem apontar para arquivos legíveis dentro dele. Reinicie após alterações.

URLs locais de ícones e o manifesto respeitam `NUXT_APP_BASE_URL`, por exemplo `/registry/`; URLs HTTP(S) externas são preservadas. Não guarde segredos nas variáveis de arquivo privado: as imagens são públicas, embora os caminhos permaneçam no servidor.
