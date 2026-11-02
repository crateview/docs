---
title: "Autenticação"
description: "Acesso anônimo, Basic Auth, sessões criptografadas e HTTPS."
---

## Autenticação suportada

Suporta acesso anônimo e htpasswd Basic Auth nativo. Serviços Bearer externos e OIDC não são suportados. Esta versão não administra usuários.

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
