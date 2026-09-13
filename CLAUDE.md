# Site de casamento — Larissa & Guilherme

Site de casamento, evento em **07/08/2027**, São José do Rio Preto/SP.
Cliente: Larissa (prima). Sem prazo apertado — meta era ficar pronto out/nov 2026.

## Regra número 1

**Nada de framework, nada de build, nada de backend.** É um site de conteúdo que
vai ao ar, fica 1 ano e morre. Laravel/Inertia/Vue/Next estão explicitamente
descartados. Se uma tarefa parecer exigir servidor, procure a solução estática
antes (link de WhatsApp, Google Forms, QR de PIX, iframe do Google Maps).

**Única exceção aberta, a pedido da Larissa:** o Mural de mensagens (seção
`#mural`) precisa guardar as mensagens de quem visita o site, o que é
impossível de forma 100% estática. Para isso usamos o **Firestore** (Firebase)
direto do navegador — sem servidor próprio, sem build, só um `<script
type="module">` chamando o SDK via CDN. Ver detalhes em "Mural de mensagens
(Firestore)" mais abaixo. Não introduza outro backend/banco de dados sem
perguntar; se precisar de mais uma feature parecida, reaproveite esse mesmo
projeto Firebase.

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
├── index.html                 # página única com âncoras (#quando, #historia, #onde, #galeria, #beleza, #presentes, #mural)
├── guia-de-beleza.html        # guia de salões/maquiadoras para madrinhas e convidadas
├── guia-de-hospedagem.html    # guia de hotéis em Rio Preto para quem vem de fora
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

## Mural de mensagens (Firestore)

Seção `#mural` em `index.html`: formulário (nome + mensagem) que grava no
Firestore e uma lista logo abaixo que mostra as últimas 50 mensagens em tempo
real, via `onSnapshot`. Tudo em `<script type="module">` no fim do `<body>`,
importando o SDK modular direto do CDN da Google (`gstatic.com/firebasejs`) —
sem npm, sem build.

**Falta fazer para o mural funcionar de verdade** (só a Larissa/o dev consegue
fazer essa parte, exige login numa conta Google):

1. Criar um projeto grátis em https://console.firebase.google.com (plano
   Spark, sem cartão de crédito).
2. Ativar o **Firestore** no modo produção.
3. Em Firestore → Regras, colar:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /mural/{mensagemId} {
         allow read: if true;
         allow create: if request.resource.data.keys().hasOnly(['nome', 'texto', 'criadoEm'])
           && request.resource.data.nome is string && request.resource.data.nome.size() <= 60
           && request.resource.data.texto is string && request.resource.data.texto.size() <= 500
           && request.resource.data.criadoEm == request.time;
         allow update, delete: if false;
       }
     }
   }
   ```

   Isso permite que qualquer visitante leia e crie mensagens, mas ninguém
   edita ou apaga mensagem de outra pessoa, e cada documento só pode ter
   exatamente os campos `nome`, `texto` e `criadoEm` dentro dos tamanhos
   definidos.
4. Em Configurações do projeto → Seus apps → adicionar um app Web → copiar o
   objeto `firebaseConfig` e colar no lugar dos `"COLE_AQUI"` no `<script
   type="module">` de `index.html`. Esse objeto **não é secreto** — é
   normal ele ficar visível no código-fonte; quem protege os dados são as
   regras do passo 3, não o `apiKey`.

**Por que Firestore e não outra coisa:** é gratuito na prática para o volume
de um mural de casamento (limite diário do plano Spark é bem maior que o
tráfego esperado), roda 100% do navegador do visitante (nada pra hospedar ou
manter no ar) e é um produto central da Google — não há sinal de
descontinuação, e times inteiros de apps de terceiros dependem dele, então a
chance de sumir ou cair antes do casamento em 08/2027 é muito baixa. O
principal risco não é o serviço cair, e sim: (a) alguém apagar o projeto
Firebase sem querer — não apague; (b) spam/abuso no formulário público — o
honeypot (`input[name="site"]` escondido) e os limites de tamanho nas regras
cobrem os casos mais comuns, mas não é 100% à prova de bot.

Enquanto o `firebaseConfig` não for preenchido, a seção mostra uma mensagem
de "mural ainda não configurado" em vez de quebrar.

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
- [x] Segundo bloco de "Nossa história" ("como tudo começou" antes do Chá Bar)
      preenchido (13/09) com o texto final da Larissa/Gui, na seção `#onde`.
- [ ] Endereço da cerimônia → ainda "Local a confirmar" (o da recepção, no
      Fauze Karam Buffet, já está preenchido com mapa e botão "Como chegar").
- [ ] Lista de presentes: Larissa avisou (12/09) que só vai definir mais pra
      frente.
- [x] Configurar o projeto Firebase do Mural de mensagens (projeto
      `casamentolarissa-e77a7`, regras já coladas, `firebaseConfig` já no
      `index.html`).
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
