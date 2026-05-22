# Check: JavaScript

JS é o vilão #1 de **TBT** e **INP**. Cada KB de JS executado bloqueia o main thread.

## Auditoria

| Item | Verificar |
|------|-----------|
| Tamanho total JS bloqueante | < 200KB ideal mobile; > 500KB → red flag |
| `defer` em scripts não-críticos | Sempre presente |
| `async` para scripts independentes | Sim para Pixel/GA4/etc, **com cuidado** |
| JS não usado | Lighthouse `unused-javascript` |
| Legacy JS (ES5 com polyfills) | `legacy-javascript` audit |
| Duplicated JS | `duplicated-javascript` (multiple jQuery, lodash full + cherry-picks) |
| Long tasks | `long-tasks` audit; > 50ms task → trava INP |
| Third-party total | `third-party-summary`; > 1.5s blocking → revisar |
| Bundle splitting | Code-split por rota? Por componente? |
| Tree-shaking | Imports default vs cherry-pick? |

## Padrão de loading

```html
<head>
  <!-- Apenas o estritamente crítico antes do </head> -->
  <script>/* tiny inline: <2KB, dataLayer init, antifraude */</script>

  <!-- Tudo mais com defer (mantém ordem, executa após DOMContentLoaded) -->
  <script defer src="/main.js"></script>
  <script defer src="/analytics-init.js"></script>
</head>

<body>
  <!-- Conteúdo -->

  <!-- Scripts não-críticos no final do body também é válido -->
  <script defer src="/chat-widget.js"></script>
</body>
```

## `defer` vs `async` (quando usar cada)

| Atributo | Ordem garantida? | Quando usar |
|----------|------------------|-------------|
| (nenhum) | sim, bloqueia | Quase **nunca** em prod |
| `defer` | sim, após HTML | Default para libs com dependências |
| `async` | **não**, dispara assim que baixa | Scripts 100% independentes (Pixel, GA, snippets isolados) |
| `type="module"` | tem defer implícito | ES modules modernos |

## Erros frequentes

| Erro | Consequência |
|------|--------------|
| jQuery + 30 plugins jQuery | 500KB+ no main thread, TBT alto |
| Bundle JS inteiro síncrono no `<head>` | Bloqueia tudo |
| Polyfills de IE11 servidos para Chrome moderno | Peso desnecessário |
| `eval()` ou Function constructor | Bloqueia JIT, lento |
| Event listeners em `scroll` sem throttle | INP > 500ms |
| `document.write` | Bloqueio crítico, deprecated |
| React/Vue rodando em página estática | Hidratação cara para nada |
| Bibliotecas inteiras importadas para 1 função | `import _ from 'lodash'` em vez de `import debounce from 'lodash/debounce'` |
| Moment.js (vs date-fns/dayjs) | Moment = 230KB; dayjs = 7KB |
| GTM disparando tags síncronas no init | INP horrível na primeira interação |

## Next.js / App Router específico

- **`"use client"`** no componente errado vira tudo em client bundle. Auditar boundary.
- **`use server` actions** OK, mas envoltas em forms.
- **Hidratação de hero** — Server Component sempre que possível.
- **`next/dynamic` com `ssr: false`** para libs que não precisam SSR (gráficos, mapas).
- **`next/script`** com `strategy="afterInteractive"` ou `"lazyOnload"` para third-party.
- **`useReportWebVitals`** para RUM nativo.

```tsx
// Boundary correto
// app/page.tsx — Server Component
import HeroForm from './HeroForm'; // Client Component só onde precisa
export default function Page() {
  return <main><h1>...</h1><HeroForm /></main>;
}

// HeroForm.tsx
'use client';
// só este componente vai pro bundle client
```

## Third-party scripts: estratégias

1. **Async com cuidado** — Pixel/GA são independentes, async funciona; mas pode perder eventos cedo.
2. **`partytown`** — roda third-party em web worker. Funciona bem com GTM, GA, Pixel. Trade-off: alguns scripts incompatíveis.
3. **GTM Server-Side** — move execução pro servidor. Melhor performance, custo extra.
4. **Lazy on user interaction** — carregar chat widget só depois do primeiro scroll/click.

## Cross-check obrigatório

1. Lighthouse `unused-javascript` → percentual e KB.
2. Lighthouse `third-party-summary` → top 3 origins por blocking time.
3. Lighthouse `long-tasks` → tasks > 50ms.
4. DevTools → Performance panel → flame chart de bootup.
5. Treemap do Lighthouse (`script-treemap-data` no JSON).
6. Conferir `Coverage` tab → % usado.

## Antes de remover qualquer biblioteca

- Grep por usos no projeto inteiro.
- Conferir se está sendo carregada por outro plugin/tema (WP).
- Conferir dependência transitiva (lib A importa lib B).
- Plano de rollback rápido.
