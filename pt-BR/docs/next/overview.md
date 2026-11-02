---
title: "Visão geral"
description: "O que o Crateview faz, como ele se conecta ao seu registro e onde estão seus limites."
---

## Uma visão mais clara do seu registro

Crateview é uma interface da web para registros de contêiner Docker e OCI. Ele ajuda você a encontrar repositórios e tags, inspecionar imagens multiplataforma, ler manifestos e configurar documentos e entender camadas compactadas e seu histórico.

Ele é executado junto com seu registro. O registro armazena imagens e impõe controle de acesso; Crateview fornece a interface e um proxy de mesma origem. Docker e outros clientes OCI podem usar a origem do frontend como endereço de registro.

![Visão geral do repositório Crateview com repositórios Alpine e de demonstração aninhados.](https://crateview.de/images/docs/next/repositories-light.webp)

## Arquitetura

```text
Browser / Docker / Podman / ORAS
              |
       Crateview (Nuxt / Nitro)
              |
   Your existing OCI registry
```

O catálogo dispensa banco; monitoramento opcional usa SQLite persistente. Contas usam htpasswd e arquivo de permissões separado.

## Recursos suportados

- Pesquisa de nome de repositório, sugestões de tags e namespaces aninhados.
- Tabelas de tags, seleção de plataforma, tamanhos de camadas compactadas e histórico de imagens.
- Anotações OCI para descrições, URLs de origem, documentação e licenças.
- Manifesto bruto e configuração JSON com suporte para área de transferência.
- Referentes OCI, incluindo o substituto de índice de tag digest padrão mantido por clientes OCI.
- Acesso anônimo e nativo htpasswd Basic Auth.
- Exclusão opcional do manifesto, com tags afetadas mostradas antes da confirmação.
- 13 idiomas de interface, Sistema/Claro/Escuro e marca configurável.

- Monitoramento opcional, desativado por padrão. Visão geral abre o painel quando ativo; caso contrário /home redireciona para /explore. A ativação expõe dados globais a todos com acesso à interface, incluindo visitantes anônimos em registros públicos. O modo token gerenciado exige leitura global adicional.

## Conheça os limites

O API padrão não fornece carimbos de data/hora push, estatísticas de download, favoritos ou verificação de vulnerabilidades. Crateview não fabrica esses valores. A data de criação de uma imagem vem dos metadados da imagem e não é o horário de upload. Os tamanhos exibidos descrevem camadas de imagem compactadas, não o uso do disco do registro.

Administração agrupa Configurações e Usuários. O menu da conta também oferece idioma e tema. Conta altera sua senha com confirmação da atual. A lista do servidor controla listagem e criação de contas; concessões de token são somente leitura. Novas senhas exigem no mínimo 12 caracteres e no máximo 72 bytes UTF-8. Novas contas de token começam sem acesso a repositórios.

A lista de permissões do servidor controla a listagem, criação, redefinição de senhas e exclusão de contas. O menu de três pontos alinhado à direita permite aos administradores redefinir a senha de outro usuário sem a senha antiga ou excluir sua conta após confirmação. A redefinição preserva as permissões dos repositórios; a exclusão as remove. Seu próprio menu abre Conta para alterar sua senha com a senha atual; a exclusão da própria conta é bloqueada.

Comece com [instalação](/pt-BR/docs/next/installation) e, em seguida, revise [configuração](/pt-BR/docs/next/configuration) e [autenticação](/pt-BR/docs/next/authentication).
