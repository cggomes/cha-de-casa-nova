# Chá de Casa Nova 🏡

Página estática de lista de presentes com contribuição via PIX. Cada presente tem uma
meta (ex.: geladeira de R$ 4.000) dividida em cotas (ex.: R$ 100), e a página gera um
**QR Code PIX com o valor já preenchido** para cada cota escolhida — além do código
"copia e cola".

Sem build, sem npm, sem backend. Três arquivos.

## Arquivos

```
index.html        a página inteira (HTML + CSS + JS + configuração)
vendor/qrcode.js  gerador de QR Code (MIT, Kazuhiko Arase) — cópia local, sem CDN
fotos/            opcional: fotos dos presentes
.nojekyll         evita que o GitHub Pages processe o site com Jekyll
```

## Como configurar

Abra `index.html` e edite o bloco `CONFIG` no início do `<script>`:

```js
const CONFIG = {
  chavePix: "123e4567-e89b-...",     // chave aleatória (EVP) gerada no app do banco
  nomeRecebedor: "CHRISTIAN E PAMELA",  // máx. 25 caracteres
  cidade: "BRASILIA",                // máx. 15 caracteres
  selo: "Chá de Casa Nova · Christian & Pâmela",
  titulo: "Nosso cantinho começa aqui",
  casal: "Christian & Pâmela",
  recado: "...",
  festa: { data: "", hora: "", endereco: "" },  // "" esconde o campo
  usarTxid: true,
};
```

Depois edite a lista `ITENS`:

```js
{
  nome: "Geladeira",
  descricao: "A peça mais importante da cozinha",
  emoji: "🧊",
  total: 4000,        // meta do item
  cota: 100,          // valor de cada cota
  txid: "GELADEIRA",  // aparece no seu extrato bancário
  foto: "",           // opcional: "fotos/geladeira.jpg"
}
```

O número de cotas é calculado sozinho (`total / cota`).

### Lojinhas (comprar direto)

Além do PIX, a página mostra uma seção com listas de presentes que você montou em lojas
online. Edite a lista `LOJAS`:

```js
{
  nome: "Nossa lista na Shopee",
  plataforma: "Shopee",            // aparece como etiqueta no cartão
  descricao: "Itens pequenos pra casa: utensílios, cama, mesa e banho.",
  emoji: "🛍️",
  url: "https://collshp.com/...",  // deixe "" pra esconder este cartão
  cor: "#EE4D2D",                  // cor da marca, usada na borda e no botão
}
```

Lojas com `url` vazia são ignoradas, e se **nenhuma** tiver link a seção inteira não
aparece — dá pra ir adicionando aos poucos sem quebrar a página.

Onde pegar o link de cada plataforma:

| Loja | Caminho |
|---|---|
| Shopee | app › `Eu` › `Minha loja`/`Lista` › `Compartilhar` › `Copiar link` (vira `collshp.com/...`) |
| Amazon | `Listas` › crie uma lista › `Enviar lista para amigos` |
| Mercado Livre | `Favoritos` › crie uma lista pública › `Compartilhar` |

Cores de marca úteis: Shopee `#EE4D2D`, Amazon `#FF9900`, Mercado Livre `#FFE600`,
Magalu `#0086FF`, Casas Bahia `#0A21C0`.

### O truque do `txid`

O campo `txid` do PIX faz o pagamento chegar **identificado no seu extrato**. Assim você
sabe quem mandou quanto e para qual presente, sem precisar de banco de dados nem de
contador na página — o extrato do seu banco é a fonte da verdade.

Use só letras e números, máx. 25 caracteres. Se algum banco reclamar do código, mude
`usarTxid` para `false`.

## Rodando localmente

É um arquivo estático — basta abrir `index.html` no navegador.

O botão "Copiar código PIX" usa a API de clipboard, que só funciona em `https://` ou
`localhost`. Abrindo via `file://` ele cai num fallback que também funciona, mas se
quiser testar igual ao ambiente real:

```sh
python3 -m http.server 8000
# abra http://localhost:8000
```

## Publicando no GitHub Pages

Sem build step, o Pages serve os arquivos direto da raiz do branch `main`:

1. `Settings` › `Pages`
2. **Source**: `Deploy from a branch`
3. **Branch**: `main` / `/ (root)` › `Save`

Em ~1 minuto o site fica em `https://<usuario>.github.io/<repo>/`.

Para atualizar depois, basta editar `index.html`, commitar e dar push.

## Nota sobre privacidade

Este projeto usa **chave aleatória** (EVP) de propósito. Gere no app do banco:
`Pix` › `Minhas chaves` › `Cadastrar chave` › `Chave aleatória`.

A chave fica visível no código-fonte da página, e isso é esperado: é justamente o que o
convidado precisa para conseguir pagar. Num repositório público ela também fica visível
no GitHub. Isso não é uma brecha — chave PIX só recebe, nunca saca —, e a chave aleatória
tem duas vantagens sobre CPF, celular ou e-mail:

- não expõe nenhum dado pessoal;
- é **descartável**: se um dia você quiser, apaga no app e gera outra, sem mexer na conta.
  Com CPF ou celular isso é impossível.

O que a chave aleatória **não** faz é tornar o pagamento anônimo: ao confirmar o PIX, o
app do pagador mostra seu nome e um CPF parcialmente mascarado, consultados no DICT. Isso
é do próprio protocolo do PIX, igual para qualquer tipo de chave.

### Por que não usar GitHub Secrets pra isso

Secrets e Environments do GitHub só existem em tempo de build. Como esta página é um
arquivo estático que roda no navegador do convidado, tudo que o QR Code precisa tem que
estar no HTML entregue. Um valor injetado por workflow acabaria dentro do `index.html`
publicado, legível em "ver código-fonte". Segredo que o cliente precisa ler não é
segredo — e chave PIX não é um segredo, é um número de conta feito pra ser distribuído.

## Licença

`vendor/qrcode.js` — MIT, © 2009 Kazuhiko Arase.
