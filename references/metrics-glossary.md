# Glossário de Métricas — PageSpeed / Lighthouse / Core Web Vitals

Referência rápida de cada métrica, o que ela mede, o que normalmente a degrada e onde verificar.

---

## Core Web Vitals (oficial Google, ranking factor)

### LCP — Largest Contentful Paint

**O que é:** tempo até o maior elemento visível na viewport ser renderizado (imagem do hero, headline grande, vídeo poster).

**Thresholds:** good ≤ 2.5s · needs improvement 2.5–4.0s · poor > 4.0s.

**Causas comuns:**
- Imagem do hero pesada, sem `fetchpriority="high"`, sem preload.
- `loading="lazy"` aplicado erroneamente no LCP element.
- Render-blocking CSS/JS no `<head>`.
- Fonte custom carregando antes do texto LCP, sem `font-display: swap`.
- TTFB alto (servidor lento, sem cache).
- Hero atrás de carrossel/slider que carrega via JS.

**Audits Lighthouse:** `largest-contentful-paint-element`, `preload-lcp-image`, `uses-optimized-images`, `uses-responsive-images`, `render-blocking-resources`, `server-response-time`.

**Onde olhar:** Performance panel → Timings → LCP. PageSpeed → "Largest Contentful Paint element".

---

### CLS — Cumulative Layout Shift

**O que é:** soma dos deslocamentos inesperados de layout durante o ciclo de vida da página (acumula até 5s de inatividade entre shifts).

**Thresholds:** good ≤ 0.1 · needs 0.1–0.25 · poor > 0.25.

**Causas comuns:**
- Imagens sem `width`/`height` (ou `aspect-ratio` em CSS).
- Iframes (YouTube, Vimeo, Panda) sem container reservado.
- Fontes custom causando FOIT/FOUT (swap muda métrica do texto).
- Banners / cookie consent inseridos depois do render.
- Ads / embeds dinâmicos.
- Botões que aparecem após hydration (Next/React).
- Animações em propriedades não-composited (top/left em vez de transform).

**Audits Lighthouse:** `layout-shift-elements`, `unsized-images`, `non-composited-animations`.

---

### INP — Interaction to Next Paint

**O que é:** maior delay entre interação (click/tap/keypress) e o próximo paint, em toda a sessão. Substituiu FID em março/2024.

**Thresholds:** good ≤ 200ms · needs 200–500ms · poor > 500ms.

**Causas comuns:**
- JS pesado no main thread (libs UI, hydration React, jQuery + plugins).
- Event handlers síncronos longos.
- GTM disparando tags síncronas no click.
- Re-renders desnecessários após interação.
- Third-party scripts bloqueando após interação (chat widgets, A/B test tools).

**Audits Lighthouse (lab proxy):** `total-blocking-time`, `mainthread-work-breakdown`, `bootup-time`, `third-party-summary`, `long-tasks`.

**Importante:** INP **só existe em field data (CrUX)**. Lighthouse lab usa TBT como proxy — mas TBT só mede main thread durante o *load*, enquanto INP fira em qualquer interação no ciclo de vida. **Pages com TBT=0 podem reprovar em INP.** Sempre cruzar.

**Componentes do INP** (otimizar os 3, não só o handler):
1. Input delay (main thread bloqueado antes do handler).
2. Processing duration (execução do handler).
3. Presentation delay (do fim do handler até o próximo frame).

**Site-level**: p75 das page views. Por page view, ignora a pior interação a cada 50 (trim p98 intra-página). Detalhes em `psi-lighthouse-internals.md`.

---

## Métricas de carregamento (lab — Lighthouse)

### FCP — First Contentful Paint

**O que é:** tempo até o primeiro pixel de conteúdo (texto, imagem, SVG) ser pintado.

**Thresholds:** good ≤ 1.8s · needs 1.8–3.0s · poor > 3.0s.

**Causas:** render-blocking, TTFB, fonts bloqueando, JS no head sem defer.

---

### TBT — Total Blocking Time

**O que é:** soma do tempo em que o main thread ficou bloqueado (>50ms tasks) entre FCP e TTI.

**Thresholds:** good ≤ 200ms · needs 200–600ms · poor > 600ms.

**Causas:** bundle JS grande, hydration, polyfills legados, third-party scripts.

**Audits:** `unused-javascript`, `duplicated-javascript`, `legacy-javascript`, `third-party-summary`.

---

### Speed Index

**O que é:** velocidade média com que o conteúdo visível é pintado (filmstrip).

**Thresholds:** good ≤ 3.4s · needs 3.4–5.8s · poor > 5.8s.

**Causas:** conteúdo do hero progressivamente revelado (skeleton, carrossel, fade-in).

---

### TTI — Time to Interactive

**O que é:** quando a página está visualmente pronta E o main thread livre por 5s.

**Status:** ainda calculada, mas **fora do score** desde Lighthouse 10. Substituída na prática por TBT/INP.

---

## Métricas de rede / servidor

### TTFB — Time to First Byte

**O que é:** tempo até o primeiro byte do HTML chegar.

**Thresholds:** good ≤ 800ms · needs 800–1800ms · poor > 1800ms.

**Causas:** hosting lento, sem cache de página (WP), PHP pesado, queries lentas, sem CDN, SSR pesado (Next).

---

## Lab vs. Field

| Aspecto | Lab (Lighthouse) | Field (CrUX) |
|---------|------------------|--------------|
| Fonte | Simulação local (Moto G4 / 4G slow) | Usuários reais Chrome (28d) |
| Métricas | FCP, LCP, TBT, CLS, SI, TTI | LCP, CLS, INP, FCP, TTFB |
| INP | ❌ não tem (usa TBT como proxy) | ✅ disponível |
| Decisão de ranking | ❌ não usado | ✅ p75 ≥ "good" em 75% dos usuários |
| Útil para | Diagnóstico, regressão CI | Priorização real, validação de fix |

**Regra:** prioriza pelo field, valida pelo lab. Score 100 lab com INP field ruim = problema real.

---

## Audits Lighthouse de alto impacto (referência rápida)

| Audit ID | Métrica | Tipicamente economiza |
|----------|---------|------------------------|
| `render-blocking-resources` | FCP, LCP | 300–2000ms |
| `uses-responsive-images` | LCP, transfer | 50–80% bytes imagem |
| `uses-optimized-images` | LCP, transfer | 30–60% bytes |
| `modern-image-formats` (WebP/AVIF) | LCP, transfer | 25–50% bytes |
| `preload-lcp-image` | LCP | 200–800ms |
| `unused-css-rules` | FCP | 50–300ms |
| `unused-javascript` | TBT, INP | 100–500ms |
| `legacy-javascript` | TBT | 50–200ms |
| `third-party-summary` | TBT, INP | varia muito |
| `font-display` | FCP, CLS | evita FOIT |
| `uses-text-compression` | FCP, transfer | 60–80% bytes texto |
| `server-response-time` | TTFB, FCP | varia |
| `efficient-animated-content` | LCP, transfer | converter GIF → vídeo |
| `unsized-images` | CLS | reduz shift |
| `non-composited-animations` | CLS, INP | reduz jank |
| `prioritize-lcp-image` | LCP | substitui `preload-lcp-image` em LH 12 |
| `lcp-discovery-insight` | LCP | novo insight audit (LH 12) — discovery do recurso LCP |
| `render-blocking-insight` | FCP, LCP | novo insight audit (LH 12) |
| `third-parties-insight` | TBT, INP | novo insight audit (LH 12) |
| `dom-size-insight` | INP, layout | DOM grande → re-renders caros |

**Pesos do Performance Score (Lighthouse 10/11/12, estáveis):** FCP 10% · SI 10% · LCP 25% · TBT 30% · CLS 25%. TTI removido do score em LH 10.
