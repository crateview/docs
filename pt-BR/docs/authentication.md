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
- **IU:** os usuários alteram sua própria senha fornecendo a senha atual. O padrão `NUXT_USER_CREATION_ALLOWED_USERS` é vazio; `alice,bob` permite que apenas esses usuários criem contas, `*` permite que todos os usuários autenticados. Esta política é aplicada no lado do servidor.
- **Permissões:** CLI concede repositórios de destino exatos, namespaces (`team/*`) ou tudo (`*`); os perfis são `reader` (pull), `writer` (pull/push) e `manager` (pull/push/excluir). O modo básico não possui funções de registro.

Ative explicitamente `NUXT_AUTH_MODE=htpasswd` ou `token` e configure o `NUXT_AUTH_SERVICE_URL` interno e o `NUXT_AUTH_SERVICE_SECRET` somente de servidor. O modo token também requer que `NUXT_AUTH_TOKEN_REALM` corresponda exatamente ao endpoint do token público. Registros anônimos e serviços de token externos não oferecem suporte ao gerenciamento de contas locais. Permissões e redefinições de senha administrativa usam a CLI auth; a UI não pode alterar as senhas de outros usuários nem atribuir permissões.

Consulte o [guia de serviço de conta](https://github.com/crateview/registry-ui/blob/main/auth/README.md) para sobreposições do Compose, configuração, migração e comandos CLI. Roteie o acesso público Docker diretamente para o registro, independentemente da IU. Os tokens emitidos anteriormente permanecem válidos por até cinco minutos após alterações de senha ou revogação de permissão.
