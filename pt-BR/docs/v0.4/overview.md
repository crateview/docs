---
title: "Visão geral"
description: "O que o Crateview faz, como ele se conecta ao seu registro e onde estão seus limites."
---

## Uma visão mais clara do seu registro

Crateview é uma interface da web para registros de contêiner Docker e OCI. Ele ajuda você a encontrar repositórios e tags, inspecionar imagens multiplataforma, ler manifestos e configurar documentos e entender camadas compactadas e seu histórico.

Ele é executado junto com seu registro. O registro armazena imagens e impõe controle de acesso; Crateview fornece a interface e um proxy de mesma origem. Docker e outros clientes OCI podem usar a origem do frontend como endereço de registro.

![Visão geral do repositório Crateview com repositórios Alpine e de demonstração aninhados.](https://crateview.de/images/docs/v0.4/repositories-light.webp)

## Arquitetura

```text
Browser / Docker / Podman / ORAS
              |
       Crateview (Nuxt / Nitro)
              |
   Your existing OCI registry
```

O catálogo dispensa banco; monitoramento opcional usa SQLite persistente.

## Recursos suportados

- Pesquisa de nome de repositório, sugestões de tags e namespaces aninhados.
- Tabelas de tags, seleção de plataforma, tamanhos de camadas compactadas e histórico de imagens.
- Anotações OCI para descrições, URLs de origem, documentação e licenças.
- Manifesto bruto e configuração JSON com suporte para área de transferência.
- Referentes OCI, incluindo o substituto de índice de tag digest padrão mantido por clientes OCI.
- Acesso anônimo e nativo htpasswd Basic Auth.
- Exclusão opcional do manifesto, com tags afetadas mostradas antes da confirmação.
- Dois idiomas de interface (inglês e alemão), Sistema/Claro/Escuro e marca configurável.


- Monitoramento opcional, desativado por padrão. Visão geral abre o painel quando ativo; caso contrário /home redireciona para /explore. A ativação expõe dados globais a todos com acesso à interface, incluindo visitantes anônimos em registros públicos.

## Conheça os limites

O API padrão não fornece carimbos de data/hora push, estatísticas de download, favoritos ou verificação de vulnerabilidades. Crateview não fabrica esses valores. A data de criação de uma imagem vem dos metadados da imagem e não é o horário de upload. Os tamanhos exibidos descrevem camadas de imagem compactadas, não o uso do disco do registro.

Os serviços externos de token Bearer, OIDC, administração de usuários e gerenciamento de chaves API não estão integrados. Os artefatos de assinatura podem ser listados como referenciadores; isso não é verificação de assinatura criptográfica.

Comece com [instalação](/pt-BR/docs/v0.4/installation) e, em seguida, revise [configuração](/pt-BR/docs/v0.4/configuration) e [autenticação](/pt-BR/docs/v0.4/authentication).
