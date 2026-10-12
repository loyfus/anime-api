# API de animes

API REST que serve os animes mais populares via a API GraphQL pública do
[AniList](https://anilist.co). Dois endpoints, sem banco: cada requisição é uma
consulta ao AniList.

![Node](https://img.shields.io/badge/Node-339933?style=flat-square&logo=node.js&logoColor=white)
![Express](https://img.shields.io/badge/Express-000000?style=flat-square&logo=express&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-F05138?style=flat-square)

Demo: [loyfus-anime-backend.vercel.app](https://loyfus-anime-backend.vercel.app)

## O problema

A GraphQL do AniList responde bem, mas força o cliente a montar query e lidar
com um envelope aninhado. Quem só quer uma lista de animes não deveria saber
que existe GraphQL.

O front do [LoyfusAnime](https://github.com/loyfus/animeFrontEnd) consome esta
API — ele nunca fala com o AniList direto.

## Decisões

**Limpar o HTML no servidor.** A descrição do AniList vem com `<br>` e `<p>`
embaralhados. O `cleanDescription` converte quebra de linha, remove as demais
tags e normaliza `` — o front recebe texto puro e não precisa montar um
parser de HTML para desenhar uma linha.

**Delegar a ordenação.** `POPULARITY_DESC` é pedido ao AniList, não aplicado
aqui. Reimplementar o ranking exigiria baixar o catálogo inteiro para discarding
os 99 que não entram.

**Erro vira resposta, não crash.** Falha do AniList devolve `500` com mensagem
em vez de derrubar o processo em serverless, onde cada requisição já é um
invólucro isolado.

## Rodar

```bash
npm install
npm start
```

## API

### `GET /`

Lista os animes mais populares.

| Parâmetro | Padrão | Para quê |
| --- | --- | --- |
| `page` | `1` | Número da página |
| `perPage` | `40` | Itens por página |

### `GET /:id`

Detalhe de um anime: título em romaji, inglês e nativo, descrição limpa,
número de episódios, gêneros, capa e onde assistir.

## Arquitetura

```
index.js              monta o app, sobe o servidor
routes/topAnimes.js   as duas rotas e a limpeza de HTML
vercel.json           deploy serverless
```

Um arquivo de rotas só, porque são duas. A limpeza de description fica ao lado
das rotas, e não em um `utils/`: é usada apenas aqui.

## Limite de taxa

Cada chamada aqui vira uma requisição ao AniList, que é pública e tem limite de
taxa. Para uso próprio isso não é problema; para republicar, respeite os
[Termos de Uso](https://anilist.co/terms) e não incremente o volume.

## Stack

| | |
| --- | --- |
| Runtime | Node · CommonJS |
| Framework | Express |
| Fonte | AniList GraphQL |
| Deploy | Vercel (serverless) |

## Licença

MIT — [LICENSE](LICENSE).
