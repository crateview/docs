---
title: "Configuração"
description: "Conexão de registro, variáveis ​​de ambiente de tempo de execução e comportamento de implantação."
---

## Camadas de configuração

Os padrões do aplicativo são definidos em `nuxt.config.ts` e `app/app.config.ts`. As variáveis ​​de ambiente substituem os padrões de tempo de execução quando o servidor é iniciado. Os comandos de desenvolvimento e construção são `.env`; o ponto de entrada do nó de produção não o lê automaticamente.

Reinicie o servidor após alterar as variáveis ​​de ambiente de tempo de execução. A edição da configuração do componente no nível de origem requer uma nova compilação de produção.

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

A configuração pública é enviada ao navegador. Nunca armazene senhas ou tokens em variáveis ​​`NUXT_PUBLIC_*`. Consulte [marca e tema](/pt-BR/docs/v0.1/theming) para opções de cores e tipografia.

## Layout de rede

O navegador se comunica com Crateview em uma origem. Nitro faz proxy `/v2/**` para clientes de registro e lê metadados de registro por meio de `/api/registry/**`. Isso evita a necessidade de acesso de navegador de origem cruzada ao próprio registro.

Atrás de um proxy reverso, preserve a origem pública e os cabeçalhos do protocolo. O endereço de front-end público também é o endereço de registro exibido nos comandos pull copiados. Para uso autenticado remotamente, encerre o HTTPS corretamente antes de aceitar as credenciais do navegador ou da CLI.

## Compose versus configuração autônoma

O arquivo Compose incluído se conecta intencionalmente ao seu próprio serviço `registry` com `NUXT_REGISTRY_UPSTREAM=http://registry:5000`. Para usar outro registro em uma implantação de contêiner, adapte esse ambiente de serviço ou execute o front-end separadamente. Não presuma que um registro de loopback de host possa ser acessado por meio de um gateway de contêiner.
