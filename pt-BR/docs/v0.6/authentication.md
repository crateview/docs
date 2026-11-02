---
title: "Autenticação"
description: "Acesso anônimo, Basic Auth, sessões criptografadas e HTTPS."
---

## Autenticação suportada

Crateview suporta registros anônimos, htpasswd Basic Auth nativo e o serviço opcional Crateview auth em modo token. Ele investiga `/v2/` e exibe o formulário de login quando o registro requer autenticação Básica.

Os serviços externos de token Bearer e OIDC não estão integrados. Os serviços de token externo não configurados produzem um aviso explícito de unsupported-auth.

## Basic Auth do lado do registro

Crie um arquivo htpasswd com um hash de senha bcrypt. O exemplo a seguir usa um contêiner Apache oficial temporário; substitua as credenciais de exemplo e proteja o arquivo resultante:

```sh
mkdir -p auth
docker run --rm --entrypoint htpasswd httpd:2   -Bbn your-user choose-a-strong-password > auth/htpasswd
```

Exemplo de configurações de Distribution:

```yaml
auth:
  htpasswd:
    realm: Registry
    path: /auth/htpasswd
```

Monte o arquivo gerado no contêiner de registro em `/auth/htpasswd`. A configuração do registro é separada da configuração Crateview. Utilize HTTPS para acesso remoto; Basic Auth é uma codificação, não uma criptografia de transporte. Evite deixar senhas reais no histórico de shell compartilhado ou em exemplos públicos.

## Sessões de IU

O servidor Nitro valida credenciais no registro. Ele os armazena em um cookie criptografado, HttpOnly, SameSite=Strict. O JavaScript do navegador e localStorage não recebem as credenciais. As sessões expiram após oito horas.

Defina um segredo de sessão de produção aleatório:

```sh
openssl rand -base64 32
```

Utilize o valor gerado como `NUXT_SESSION_PASSWORD`, com no mínimo 32 caracteres. Cada instância de front-end por trás de um balanceador de carga deve usar o mesmo valor. Sem configuração, Crateview gera um segredo local do processo, portanto, uma reinicialização invalida as sessões existentes.

## HTTPS e logout

O cookie de sessão usa Secure quando o protocolo de solicitação pública é HTTPS. Configure corretamente o protocolo encaminhado do proxy reverso. Sair limpa a sessão da UI; ele não exclui nem modifica usuários do registro.

Docker e outros clientes CLI são autenticados diretamente por meio do proxy `/v2/**` usando suas credenciais de registro normais. O login na UI da web não faz login na CLI Docker automaticamente.

## Gerenciamento de conta opcional

Um serviço de gerenciamento/auth independente oferece suporte à criação de contas e alterações de senha própria. Docker se conecta diretamente ao registro e funciona sem o contêiner de UI. Os usuários e os hashes bcrypt permanecem em `htpasswd`; permissões de token residem em `permissions.json`. O armazenamento da conta não usa SQLite.

- **Modo básico:** Distribution verifica o arquivo htpasswd compartilhado; todos os usuários têm igual acesso ao registro. Docker não requer a execução do serviço de gerenciamento.
- **Modo token:** O serviço independente auth emite tokens assinados de cinco minutos; Distribution impõe permissões de repositório. Docker requer o registro e o serviço auth. Novos usuários inicialmente não têm acesso ao repositório.
- **UI:** A lista de permissões do servidor controla a listagem, criação, redefinição de senhas e exclusão de contas. `NUXT_USER_CREATION_ALLOWED_USERS`: `alice,bob`; `*`. Um valor vazio nega a administração; os nomes listados são permitidos e o curinga permite todos os usuários autenticados.
- **Permissões:** CLI concede repositórios de destino exatos, namespaces (`team/*`) ou tudo (`*`); os perfis são `reader` (pull), `writer` (pull/push) e `manager` (pull/push/excluir). O modo básico não possui funções de registro.

Ative explicitamente `NUXT_AUTH_MODE=htpasswd` ou `token` e configure o `NUXT_AUTH_SERVICE_URL` interno e o `NUXT_AUTH_SERVICE_SECRET` somente de servidor. O modo token também requer que `NUXT_AUTH_TOKEN_REALM` corresponda exatamente ao endpoint do token público. Registros anônimos e serviços de token externos não oferecem suporte ao gerenciamento de contas locais. As permissões dos repositórios são gerenciadas pela CLI de autenticação; a interface as exibe somente para leitura.

Consulte o [guia de serviço de conta](https://github.com/crateview/registry-ui/blob/8fe2c00e9419faf971c2ab66fa3a561cb3dc6c00/auth/README.md) para sobreposições do Compose, configuração, migração e comandos CLI. Roteie o acesso público Docker diretamente para o registro, independentemente da IU. Os tokens emitidos anteriormente permanecem válidos por até cinco minutos após alterações de senha ou revogação de permissão.

Administração agrupa Configurações e Usuários. O menu da conta também oferece idioma e tema. Conta altera sua senha com confirmação da atual. A lista do servidor controla listagem e criação de contas; concessões de token são somente leitura. Novas senhas exigem no mínimo 12 caracteres e no máximo 72 bytes UTF-8. Novas contas de token começam sem acesso a repositórios.

A lista de permissões do servidor controla a listagem, criação, redefinição de senhas e exclusão de contas. O menu de três pontos alinhado à direita permite aos administradores redefinir a senha de outro usuário sem a senha antiga ou excluir sua conta após confirmação. A redefinição preserva as permissões dos repositórios; a exclusão as remove. Seu próprio menu abre Conta para alterar sua senha com a senha atual; a exclusão da própria conta é bloqueada.

![Versão da documentação](https://crateview.de/images/docs/v0.6/users-light.webp)

![Ações](https://crateview.de/images/docs/v0.6/user-actions-light.webp)

![Alterar a senha](https://crateview.de/images/docs/v0.6/user-password-light.webp)

![Excluir usuário](https://crateview.de/images/docs/v0.6/user-delete-light.webp)

```dotenv
NUXT_AUTH_MODE=htpasswd
NUXT_AUTH_SERVICE_URL=http://auth:8080
NUXT_AUTH_SERVICE_SECRET=REPLACE_WITH_SEPARATE_MANAGEMENT_SECRET
NUXT_USER_CREATION_ALLOWED_USERS=alice,bob
```

![Versão da documentação](https://crateview.de/images/docs/v0.6/account-light.webp)

![Versão da documentação](https://crateview.de/images/docs/v0.6/login-light.webp)
