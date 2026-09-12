# Site de casamento — Larissa & Guilherme

Site de casamento, evento em **07/08/2027**, São José do Rio Preto/SP.
Cliente: Larissa (prima). Sem prazo apertado — meta era ficar pronto out/nov 2026.

## Regra número 1

**Nada de framework, nada de build, nada de backend.** É um site de conteúdo que
vai ao ar, fica 1 ano e morre. Laravel/Inertia/Vue/Next estão explicitamente
descartados. Se uma tarefa parecer exigir servidor, procure a solução estática
antes (link de WhatsApp, Google Forms, QR de PIX, iframe do Google Maps).

## Stack

| Camada | Escolha |
|---|---|
| Marcação | HTML5 estático, escrito à mão |
| Estilo | CSS puro em `<style>` no `<head>`, custom properties em `:root` |
| JS | Vanilla, IIFE no fim do `<body>`. Hoje só a contagem regressiva |
| Fontes | Google Fonts (Playfair Display, Cormorant Garamond, Alex Brush) |
| Build | Nenhum |
| Dependências | Zero. Não adicionar npm, bundler, Tailwind ou CDN de framework |

Se algum dia o número de páginas passar de ~6 e a duplicação de header/footer
incomodar, a única migração aceitável é **Astro** (continua gerando estático).
Não migre por conta própria — pergunte antes.

## Estrutura do repositório

```
/
├── index.html                 # página única com âncoras (#quando, #historia, #onde, #galeria, #beleza, #presentes)
├── guia-de-beleza.html        # guia de salões/maquiadoras para madrinhas e convidadas
├── fotos/
│   ├── porsol.jpg             # capa (hero)
│   ├── flores.jpg
│   ├── barco.jpg
│   ├── vestido-vermelho.jpg
│   ├── vestido-verde.jpg
│   ├── estadio.jpg
│   ├── carnaval-1.jpg
│   └── carnaval-2.jpg
└── CLAUDE.md
```

Não crie `src/`, `public/`, `assets/` nem pastas de config. A raiz do repo é a
raiz do site publicado.

## Design

Herdado do "Guia de Beleza" e do Save the Date que a Larissa já aprovou.
Tokens já definidos em `index.html`:

```
--cream     #f5f1e6   fundo principal
--cream-2   #efe9d8   fundo de seção alternada
--olive     #57603f   texto de apoio, botões
--olive-dark#3d4530   títulos
--gold      #a9884f   filetes, rótulos
--gold-light#cfb87c   bordas
--sand      #e8dcc0   texto claro sobre olive
--ink       #3a3a30   corpo de texto
--white     #fdfaf2   texto sobre foto
```

Tipografia: `Playfair Display` (títulos, rótulos em caixa alta com
letter-spacing), `Cormorant Garamond` (corpo, 20px), `Alex Brush` (nomes do
casal, só isso).

Regras de layout:

- **Mobile é o principal, não o "também".** O shell tem `max-width: 560px`
  centralizado — no desktop o site é uma coluna, de propósito. Não faça
  layout de duas colunas para telas grandes.
- Títulos de seção alinhados à esquerda, com um filete dourado de 48px abaixo.
  Não usar "olho" em caixa alta acima de cada título (fica com cara de template).
- Alvos de toque com no mínimo 44px de altura.
- Contraste mínimo 4.5:1. Texto claro sobre o olive usa `--sand` (#e8dcc0),
  nunca `--gold-light`.
- O wrapper usa `overflow-x: clip`, **não** `hidden` — `hidden` cria um scroll
  container e quebra o `position: sticky` do menu.
- Números da contagem: `font-variant-numeric: tabular-nums lining-nums`, senão
  a largura pula a cada segundo.

## Hospedagem

**Cloudflare Pages** (escolhido). Grátis, SSL automático, CDN global, deploy a
cada push, banda ilimitada na prática.

Setup:

1. Push do repo para o GitHub.
2. Cloudflare Dashboard → Workers & Pages → Create → Pages → Connect to Git.
3. Build command: **vazio**. Build output directory: **`/`** (raiz).
4. Deploy. Sai em `<projeto>.pages.dev`.

Alternativas equivalentes, caso algo trave: Netlify (`.netlify.app`) ou
GitHub Pages (`.github.io`, basta ativar em Settings → Pages → branch `main`).

## Domínio

Ainda **não decidido** pela noiva. Ordem de preferência:

1. **`larissaeguilherme.pages.dev`** — grátis, sai junto com o deploy, zero
   configuração. É o padrão até ela pedir outra coisa.
2. **`.com.br` no registro.br** — ~R$ 40/ano, é o único registrador oficial
   brasileiro. Requer CPF. Depois de registrar: no registro.br trocar os
   servidores DNS para os da Cloudflare, e na Cloudflare Pages adicionar o
   Custom Domain. SSL sai sozinho.
3. **`.com` em Cloudflare Registrar** — ~R$ 60/ano, vendido a preço de custo e
   já integrado.

Domínios "grátis" de verdade (eu.org, no-ip, is-a.dev) não valem a pena aqui:
aprovação demorada, aparência amadora num convite de casamento, e risco de
expirar no meio do caminho. Se a ideia é não gastar, fique no `.pages.dev`.

## Decisões já tomadas (não reabrir sem perguntar)

- **Sem RSVP no site.** Confirmação de presença é por WhatsApp, fora do site.
- **Sem CMS.** Quem edita os textos é o desenvolvedor, direto no HTML.
- **Sem lista de presentes por enquanto.** A seção existe com texto de espera.
  Quando definirem: se for PIX, é chave + QR estático (imagem no repo, nada de
  gerador em JS). Se virarem cotas com pagamento, usar links do Mercado Pago —
  continua sem backend.
- Página única com rolagem + o guia de beleza como página separada.

## Pendências

- [ ] Confirmar a data: o Save the Date diz 07/08/2027, confirmar com a noiva.
- [ ] Textos reais de "Nossa história" e "O pedido" (hoje são placeholders).
- [ ] Endereço da cerimônia e da recepção → substituir os blocos
      "Local a confirmar" e trocar a caixa tracejada por um iframe do Google
      Maps (`loading="lazy"`) + botão "Como chegar" apontando para
      `https://www.google.com/maps/dir/?api=1&destination=<endereço>`.
- [ ] Guia de beleza: link direto do Instagram da "Bia Maquiadora" (hoje aponta
      para uma busca) e valores da Priscila Larsen Salon.
- [ ] Otimizar as fotos: as JPEGs vieram do WhatsApp em tamanho original.
      Redimensionar para no máximo 1400px no maior lado e gerar `.webp` com
      `<picture>`. Adicionar `loading="lazy"` em tudo menos na foto do hero.
- [ ] Favicon e imagem de compartilhamento (`og:image`) definitivos.

## Como trabalhar aqui

- Abra `index.html` direto no navegador para testar. Não precisa de servidor;
  se quiser um, `python3 -m http.server`.
- Teste sempre em largura de 375px antes de qualquer outra coisa.
- Mudou texto? Um commit. Mudou layout? Confira o sticky do menu e o contraste
  do rodapé, que já quebraram uma vez.
- Escreva em português do Brasil, inclusive comentários e commits.
