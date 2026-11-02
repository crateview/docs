---
title: "Usando Crateview"
description: "Navegue pelas imagens, troque de plataforma e inspecione os dados por trás de uma tag."
---

## Encontre repositórios e tags

A visão geral lista repositórios do catálogo de registro. Explore nomes de repositórios de filtros; uma consulta `repository:tag` pesquisa tags nesse repositório. Nomes aninhados como `team/service` são suportados.

Pressione `/` fora de um campo editável, ou `Ctrl+K`, para focar a pesquisa na barra de navegação. A barra lateral pode ser recolhida no botão de alternância ao lado da pesquisa. Os controles de idioma e modo de cor ficam na parte inferior da barra lateral expandida.

## Inspecionar uma imagem

Abra um repositório e selecione uma tag. A visualização da imagem mostra seu digest, plataformas disponíveis, tamanhos de camada compactada e metadados de criação.

Para uma imagem multiplataforma, escolha o sistema operacional e a arquitetura no seletor de plataforma. O histórico da camada e a configuração da imagem selecionada seguem essa escolha. A guia Manifesto bruto mostra o manifesto ou índice raiz da imagem; Configuração mostra a configuração da plataforma selecionada.

![Detalhes da imagem Alpine com seleção de plataforma, histórico de camadas e comando pull.](https://crateview.de/images/image-light.webp)

## Comandos Pull

O painel de comando fornece comandos Docker, Podman e Skopeo para imagens e comandos de artefato ORAS/Podman para artefatos. Copie o comando e execute-o em seu próprio terminal.

```sh
docker pull registry.example.com/team/service:latest
podman pull registry.example.com/team/service:latest
oras pull registry.example.com/team/report:latest
```

Substitua o host de exemplo pela sua origem pública Crateview. A autenticação CLI é separada da sessão do navegador.

## Metadados e referenciadores OCI

Descrições, licenças e links de origem são lidos nas anotações OCI ou rótulos de configuração de imagem, quando disponíveis. Os metadados ausentes permanecem vazios em vez de serem adivinhados.

A guia Referenciadores lê os referenciadores OCI API. Se o registro não implementar esse endpoint, Crateview lê o índice de tag derivado de digest padrão mantido pelos clientes OCI. Ele não cria seu próprio índice verificando cada manifesto. Os artefatos de assinatura listados não são verificados automaticamente ou marcados como confiáveis.

## Interpretar carimbos de data/hora e tamanhos

O tempo de criação da imagem é fornecido pela configuração ou pelas anotações da imagem. Ele não informa quando uma tag foi enviada para esse registro. Os tamanhos das camadas compactadas não medem o consumo do disco, porque as camadas podem ser compartilhadas e os dados excluídos podem permanecer até a coleta de lixo.
