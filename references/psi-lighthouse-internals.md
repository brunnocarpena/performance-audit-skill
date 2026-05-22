# PSI / Lighthouse / CrUX — Internals e mecânica detalhada

Referência profunda sobre como PageSpeed Insights, Lighthouse e CrUX funcionam por dentro. Consultar quando precisar explicar discrepâncias, calibrar expectativas, ou integrar PSI via API.

Fontes primárias: web.dev, developer.chrome.com, github.com/GoogleChrome/lighthouse, developers.google.com.
Última revisão: 2026-05.

---

## 1. Performance Score — pesos e curva

Pesos estáveis em **Lighthouse 10, 11 e 12** (última mudança foi LH 10, quando TTI saiu e CLS subiu de 15% → 25%):

| Métrica | Peso |
|---------|------|
| FCP | 10% |
| SI  | 10% |
| LCP | 25% |
| TBT | 30% |
| CLS | 25% |
| TTI | removido |

**Curva de pontuação:** log-normal ancorada no HTTP Archive. O site no percentil 25 mapeia para score 50; percentil 8 mapeia para score 90. Entre 0.50–0.92 a relação é quase linear; acima de 0.92 há retornos drasticamente decrescentes (subir de 92 → 99 custa muito mais do que de 70 → 85).

Fonte: developer.chrome.com/docs/lighthouse/performance/performance-scoring

---

## 2. INP — mecânica completa

### Cálculo
- **Por página**: ignora a interação mais alta a cada 50 interações (trimming de outliers ≈ 2%, p98 *intra-página*).
- **Por site**: o valor reportado em CrUX é o **p75 das page views** (não p98). Confusão comum: o p98 se aplica à seleção da pior interação *dentro de uma única page view*.

### O que conta como interação
- ✅ Click (mouse), tap (touch), keypress (teclado).
- ❌ Scroll, hover, zoom — **não contam** para INP.

### Três componentes
1. **Input delay** — tempo antes do event handler começar (main thread bloqueado).
2. **Processing duration** — tempo executando os callbacks do handler.
3. **Presentation delay** — do fim do callback até o próximo frame ser pintado.

Otimizar INP exige atacar os três. Reduzir só o handler (processing) não resolve se o presentation delay (re-render React, layout thrash) for o gargalo.

### Thresholds (p75)
Good ≤ 200ms · Needs Improvement 201–500ms · Poor > 500ms.

### Soft Navigation INP (SPAs)
- **Heurística**: interação iniciada por usuário → mudança visível de URL (History API) → paint resultante.
- **Status (2026-05)**: ainda **experimental / origin trial**. OTs em Chrome 110/117/124/139, "hopefully final" planejado para Chrome 147–149.
- **Modelo de medição**: INP reseta a cada soft-nav boundary. Cada "página virtual" tem seu próprio INP.
- CrUX **ainda não reporta** soft-nav CWV. Para SPAs medir agora: instrumentar `web-vitals.js` com flag de soft-nav ou habilitar `chrome://flags/#soft-navigation-heuristics`.

Fontes: web.dev/articles/inp · developer.chrome.com/docs/web-platform/soft-navigations-experiment

---

## 3. CrUX — granularidade e disponibilidade

### Agregação
| Nível | CrUX API | PSI | BigQuery mensal | CrUX Dashboard |
|-------|----------|-----|------------------|-----------------|
| Origin | ✅ | ✅ (fallback) | ✅ | ✅ |
| URL    | ✅ | ✅ (preferido) | ❌ (origin-only) | ❌ |

- **Janela**: 28 dias rolling (CrUX API, PSI). BigQuery: snapshots mensais (publicados ~2ª terça do mês seguinte).
- **Elegibilidade**: origin/URL precisa ser publicamente descobrível, indexável, e ter amostras suficientes. Threshold exato não é divulgado. Se >20% do tráfego cai em buckets inelegíveis (ex: um único device form factor), o registro some.
- **Normalização de URL**: query string e fragmento são removidos antes de agregar (`?utm_*` e `#anchor` colapsam para a URL base).
- **PSI fallback**: PSI pede CrUX page-level; se insuficiente, cai para `originLoadingExperience`; se ambos faltarem, omite a seção field.

Fontes: developer.chrome.com/docs/crux/methodology · developer.chrome.com/docs/crux/methodology/metrics

---

## 4. PageSpeed Insights API v5

### Endpoint
```
GET https://www.googleapis.com/pagespeedonline/v5/runPagespeed
```

### Parâmetros
| Param | Valores | Notas |
|-------|---------|-------|
| `url` | URL pública | obrigatório |
| `strategy` | `desktop` (default oficial) / `mobile` | ⚠️ default é `desktop`. Para mobile-first audit, **sempre passar explícito** `strategy=mobile` |
| `category` | `performance`, `accessibility`, `best-practices`, `seo` | repetível; **`pwa` removido em LH 12 (2025)** |
| `locale` | `pt-BR`, `en-US`, ... | afeta strings do report |
| `key` | API key GCP | opcional mas crítico — sem ela pode disparar fluxo de captcha (ver `captchaResult`) |
| `captchaToken` | string | usado em resposta a `captchaResult: CAPTCHA_NEEDED` |
| `utm_source`/`utm_campaign` | string | analytics |

### Rate limits
- **Sem API key**: keyed por IP, ~1 QPS, poucas centenas/dia, 429 frequente, **pode disparar captcha**.
- **Com API key**: a doc oficial PSI v5 **não publica QPS/dia atualmente**. Valores históricos comunitários ~400 QPS por 100s e ~25.000 queries/dia. **Sempre verificar no Cloud Console** (APIs & Services → Quotas) por projeto — Google ajusta sem aviso prévio.
- **CrUX API** tem quota separada (~25k/dia default, ajustável).

### 🚨 Deprecação anunciada (ago/2025) — CrUX dentro do PSI vai sair

A documentação `get-started` (atualizada 28-ago-2025) diz textualmente:

> "We plan to discontinue including real-world data from the Chrome User Experience Report in this API. We recommend the CrUX API or the CrUX History API instead."

**Impacto na skill:**
- Os campos `loadingExperience` e `originLoadingExperience` do PSI **serão removidos** em data futura não anunciada.
- A partir dessa transição, PSI fica **só com lab data** (Lighthouse).
- **Migrar para CrUX API** para field data: `https://chromeuxreport.googleapis.com/v1/records:queryRecord` (e `queryHistoryRecord` para histórico).
- Monitorar: developer.chrome.com/docs/crux/api · developers.google.com/speed/docs/insights/v5/about

### Estrutura de resposta (top-level)
- `kind`: `"pagespeedonline#result"` (constante).
- `captchaResult`: enum — `CAPTCHA_BLOCKING`, `CAPTCHA_NEEDED`, `CAPTCHA_NOT_NEEDED`, `CAPTCHA_MATCHED`, `CAPTCHA_UNMATCHED`.
- `lighthouseResult` — JSON Lighthouse lab completo:
  - `categories.performance.score`
  - `audits[*]` com `id`, `title`, `score`, `scoreDisplayMode`, `displayValue`, `details`
  - `categories[*].auditRefs[]` com `{id, weight, group}` (permite recalcular score ponderado local)
  - `configSettings` (`emulatedFormFactor`, `locale`, `onlyCategories`)
  - `environment` (`networkUserAgent`, `hostUserAgent`, `benchmarkIndex`)
  - `timing.total`, `runWarnings[]`, `i18n`
  - **`runtimeError`** — `{code, message}` populado quando Lighthouse rodou mas a página falhou (ex.: `NO_FCP`, `ERRORED_DOCUMENT_REQUEST`, `FAILED_DOCUMENT_REQUEST`). **HTTP é 200 mesmo com runtimeError — checar explicitamente em CI.**
- `loadingExperience` — CrUX field page-level (será deprecated):
  - `metrics.{FIRST_CONTENTFUL_PAINT_MS, LARGEST_CONTENTFUL_PAINT_MS, CUMULATIVE_LAYOUT_SHIFT_SCORE, INTERACTION_TO_NEXT_PAINT, EXPERIMENTAL_TIME_TO_FIRST_BYTE}`
  - Cada métrica: `{percentile, category: FAST|AVERAGE|SLOW|NONE, distributions[]: {min, max, proportion}}`
  - `overall_category` no nível pai
- `originLoadingExperience` — mesmo shape, origin-level (fallback; será deprecated).
- `analysisUTCTimestamp`, `id`, `version`.

### `scoreDisplayMode` valores oficiais
`binary`, `numeric`, `informative`, `manual`, `not_applicable`, `error`. Score `null` é válido quando mode é `manual`/`informative`/`not_applicable`.

### `audits[*].details.type` (não enumerado pela API)
Não é contrato estável — vem do schema do Lighthouse e muda entre versões. Valores comuns observados: `opportunity`, `table`, `debugdata`, `criticalrequestchain`, `filmstrip`, `screenshot`, `treemap-data`, `list`.

### Exemplo prático (extrair só o essencial)
```bash
curl -s "https://www.googleapis.com/pagespeedonline/v5/runPagespeed?url=https://exemplo.com&strategy=mobile&category=performance&key=$PSI_KEY" \
| jq '{
    score: .lighthouseResult.categories.performance.score,
    lab: .lighthouseResult.audits | with_entries(select(.key | IN("largest-contentful-paint","cumulative-layout-shift","total-blocking-time","first-contentful-paint","speed-index","server-response-time")) | .value |= .displayValue),
    field: .loadingExperience.metrics,
    overall_field: .loadingExperience.overall_category
  }'
```

### Erros / HTTP codes (não enumerados oficialmente — comportamento observado)
- **400** — `url` ausente/malformado, `strategy` inválido.
- **403** — key inválida, API não habilitada no projeto, referer restriction violada.
- **429** — quota excedida (QPS ou daily).
- **500** — falha interna; **retry com backoff exponencial** (1s → 2s → 4s + jitter).
- **200 + `lighthouseResult.runtimeError`** — analisar antes de declarar sucesso (gotcha clássico em CI/CD).

Formato de erro padrão Google: `{error: {code, message, errors:[{domain, reason, message}]}}`.

### Autenticação
- `key=` na query é o método oficial. PSI **não exige OAuth**.
- Header `Authorization: Bearer <token>` não é necessário.
- Para keys embarcadas no front-end, restringir por referer/IP no Cloud Console.

### Pricing
- **PSI API**: gratuita, sem tier pago.
- **CrUX API**: gratuita.
- **CrUX BigQuery**: dataset gratuito, mas queries cobram taxa padrão BigQuery (~$6.25/TiB scaneado, primeiro 1 TiB/mês grátis). Cuidado com `SELECT *` em tabelas mensais (TBs).
- **pagespeed.web.dev** (UI): gratuita, usa a mesma API.

### Limitações de URL
- Páginas devem ser **públicas (crawlable + indexável)**.
- ❌ Não auditável: localhost, `127.0.0.1`, IPs privados, URLs atrás de basic-auth ou cookie-auth, redirects internos.
- Páginas com `noindex` parcial **funcionam no lab** mas **sem CrUX field data**.
- URLs com query strings são distintas (afeta agregação CrUX que normaliza).

### CI/CD best practices
- Sempre passar `key=` (evita `captchaResult` bloqueante e usa quota do projeto).
- **Checar `lighthouseResult.runtimeError` antes do `score`** — score `null` ou inválido pode passar despercebido.
- Rodar `strategy=mobile` e `strategy=desktop` em paralelo (chamadas separadas; endpoint não combina).
- Para Core Web Vitals reais, **migrar para CrUX API** antes da descontinuação no PSI.
- Cache de resposta: `analysisUTCTimestamp` permite dedupe; CrUX atualiza diariamente, então TTL ~24h é seguro.
- Retries: 500/429 com backoff; 400/403 são determinísticos, não retentar.

Fontes: developers.google.com/speed/docs/insights/v5/about · developers.google.com/speed/docs/insights/v5/get-started · developers.google.com/speed/docs/insights/v5/reference/pagespeedapi/runpagespeed · developer.chrome.com/docs/crux/api

---

## 5. Lab × Field — discrepâncias canônicas

| Métrica | Por que diferem |
|---------|------------------|
| **LCP** | Lab: cold cache, mid-tier throttled, 1 viewport, 1 geo. Field: returning visitors (warm cache, bfcache), viewports variados, elementos LCP variados, AMP/SXG preloads, **para de medir no primeiro scroll/input**. → Lab tende a ser **pessimista para cached/returning** e **otimista para conexões piores que slow-4G**. |
| **INP** | Lab **não tem INP**. TBT é proxy fraco: só mede main-thread blocking durante *load inicial*; INP fira em qualquer interação no ciclo de vida da página. **Pages com TBT=0 podem reprovar em INP.** |
| **CLS** | Lab: só durante o load trace. Field: **sessão inteira até page hide**. Lazy images sem dimensão, sticky headers que mudam ao scroll, consent banners pós-load → só aparecem em field. |
| **TTFB** | Lab: 1 location, throttled. Field: agrega geos diversas, cache de CDN warm/cold. |

Fonte: web.dev/articles/lab-and-field-data-differences

---

## 6. Throttling

### Padrão mobile (Slow 4G)
- RTT: **150ms**
- Downlink: **1.6 Mbps** (1638.4 Kbps)
- Uplink: **750 Kbps**
- CPU: **4× slowdown**

### Modos
- **Simulated** (default PSI e `lighthouse` CLI): carrega sem throttling real, Lantern simula timeline throttled a partir das dependências observadas. Rápido, baixa variância, mas inexato quando o caminho de execução muda sob carga.
- **Applied / DevTools** (`--throttling-method=devtools`): throttling real de CPU e rede durante o run. Mais preciso, mais lento, mais variância.

### Device
- Label no relatório ainda diz **"Emulated Moto G4"** — bug cosmético conhecido (issue #12297).
- CPU target **foi atualizado** para classe Moto G Power. BenchmarkIndex alvo subiu em LH 11.0. Mantém o 4× multiplier.

Fontes: github.com/GoogleChrome/lighthouse/blob/main/docs/throttling.md · debugbear.com/blog/simulated-throttling

---

## 7. Mudanças recentes (2024–2026)

- **Mar 2024**: INP substitui FID oficialmente como CWV.
- **2024–2025**: PSI passa a expor **LCP subpart breakdown** via CrUX (TTFB / resource-load-delay / resource-load-time / element-render-delay). Surface principalmente na CrUX API; ainda discreto na UI do PSI.
- **2025**: Lighthouse adota **Insights audits** compartilhados com o painel Performance do DevTools — ex: `lcp-discovery-insight`, `render-blocking-insight`, `third-parties-insight`, `prioritize-lcp-image` (substitui `preload-lcp-image`), `dom-size-insight`. Audits clássicos sendo de-emphasized.
- **2025** refinamentos:
  - TTFB passa a contar respostas Early Hints (`103`).
  - LCP melhor em imagens cross-origin (sem `Timing-Allow-Origin` ainda há censura parcial).
  - INP não conta mais drag de seleção de texto que só dispara scroll.
- **Browser parity**: **Firefox 144 (out/2025)** ship INP; Safari TP com LCP+INP em progresso. CrUX permanece Chrome-only.
- **Thresholds CWV**: **inalterados** (LCP 2.5s / CLS 0.1 / INP 200ms). ⚠️ Blogs SEO afirmando "thresholds 2026 mais rigorosos (LCP 2.0s, CLS 0.08, INP 150ms)" são **especulação, não confirmados por Google**.

Fontes: web.dev/blog/inp-cwv · debugbear.com/blog/2025-in-web-performance

---

## 8. Limites, gotchas e variância

- **PSI score ≠ Lighthouse local** mesmo na mesma versão. PSI roda no hardware do Google em datacenter fixo (US-Central). Pages TBT-heavy têm os maiores gaps. Comparar sempre na mesma superfície.
- **Variância run-to-run**: simulated é mais estável que applied, mas ±5–10 pontos em TBT/SI é normal. **Rodar 3–5× e pegar mediana**, ou usar `lighthouse-ci`.
- **Mobile vs desktop** são famílias de score separadas. Não comparar diretamente.
- **Audits removidos / renomeados**:
  - `first-meaningful-paint` (FMP) — removido.
  - `estimated-input-latency` — removido (substituído por TBT).
  - `time-to-interactive` — calculado mas **fora do score** desde LH 10.
  - `preload-lcp-image` → substituído por `prioritize-lcp-image`.
  - Categoria PWA — deprecated/removed em LH 12.
- **Field section ausente no PSI**: tipicamente amostras CrUX insuficientes. Cair para `originLoadingExperience` ou consultar CrUX API direto.
- **CLS field noise**: sites com <~1k sessões/dia têm CWV ruidoso; uma sessão muito janky pode dominar p75.
- **INP pré-prod**: não existe equivalente lab confiável. Instrumentar `web-vitals.js` em fluxos sintéticos. **Não confiar só em TBT.**

Fontes: web.dev/articles/inp · debugbear.com/docs/metrics/lighthouse-performance · unlighthouse.dev/learn-lighthouse/pagespeed-insights-vs-lighthouse

---

## 9. Heurísticas de interpretação rápida

Quando bater os números, decidir prioridade:

| Cenário | Leitura | Ação |
|---------|---------|------|
| Lab LCP bom, Field LCP ruim | Cache mascara lab; usuários reais sofrem | **Priorizar** — investigar TTFB field, hero discovery |
| Lab LCP ruim, Field LCP bom | Returning visitors com cache; lab pessimista | Não despriorizar mas baixar urgência |
| Lab TBT 0, Field INP poor | Interações pós-load (handlers, hydration tardia) | Investigar event handlers, third-party reativo |
| CLS lab 0, CLS field >0.1 | Banner/consent/ads injetados no real | Auditar consent flow, reservar espaço |
| PSI sem field | Tráfego insuficiente | Instalar RUM (`web-vitals`); usar lab + origin fallback |
| Score 92→85 entre runs | Variância normal de simulated throttling | Rodar 5× mediana antes de concluir regressão |
