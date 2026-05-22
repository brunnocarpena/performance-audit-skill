# Coleta de Dados — Performance Audit

Como reunir evidência **antes** de auditar.

A regra é a mesma da `congruence`: nenhuma análise sem evidência. Para performance, a evidência vem de quatro fontes — **lab, field, código e rede**.

---

## 1. Lab data (Lighthouse)

### Lighthouse local

```bash
# CLI (Node 18+)
npx lighthouse https://exemplo.com --output=json --output-path=./lh.json --form-factor=mobile
npx lighthouse https://exemplo.com --output=json --output-path=./lh-desktop.json --preset=desktop

# Chrome DevTools → Lighthouse panel (mais simples para sessão única)
```

**Sempre rodar 3× e usar a mediana.** Lab é simulação, varia entre runs.

### Lighthouse via PageSpeed Insights API

**Setup da chave (uma vez):**
1. Em https://console.cloud.google.com/apis/credentials → criar API key.
2. Habilitar **PageSpeed Insights API** e **Chrome UX Report API** em APIs & Services → Library.
3. Exportar no `~/.zshrc`:
   ```bash
   export PSI_API_KEY="AIzaSy..."
   export CRUX_API_KEY="$PSI_API_KEY"
   ```

Sem chave: ~1 QPS keyed por IP, cota diária baixíssima → 429 rápido.
Com chave: 400 QPS / 25.000 queries por dia.

```bash
curl "https://www.googleapis.com/pagespeedonline/v5/runPagespeed?url=https://exemplo.com&strategy=mobile&category=performance&key=${PSI_API_KEY}" \
  > psi-mobile.json
curl "https://www.googleapis.com/pagespeedonline/v5/runPagespeed?url=https://exemplo.com&strategy=desktop&category=performance&key=${PSI_API_KEY}" \
  > psi-desktop.json
```

Extrai automaticamente lab + field (CrUX).

### Chrome DevTools MCP (se disponível)

`devtools-mcp` (v0.19+) expõe traces de performance e Lighthouse direto via tool. Preferir quando disponível — dá acesso a long tasks, paint timings, etc.

---

## 2. Field data (CrUX)

CrUX só existe se a página tiver tráfego suficiente para o Chrome reportar.

### CrUX API

```bash
curl -X POST "https://chromeuxreport.googleapis.com/v1/records:queryRecord?key=${CRUX_API_KEY}" \
  -H 'Content-Type: application/json' \
  -d '{
    "url": "https://exemplo.com/",
    "formFactor": "PHONE",
    "metrics": ["largest_contentful_paint","cumulative_layout_shift","interaction_to_next_paint"]
  }'
```

### CrUX BigQuery (histórico)

Tabela pública `chrome-ux-report.all.YYYYMM` — útil para tendências de 6+ meses.

### CrUX Dashboard (Looker Studio)

[g.co/chromeuxdash](https://g.co/chromeuxdash) — gráfico mensal por origin. Bom pra investigação de "quando piorou?".

> **Sem CrUX disponível?** Marcar "sem dados de campo" no relatório e basear-se em lab. Sugerir implementar RUM (web-vitals lib).

---

## 3. RUM (campo real, primeira parte)

Quando o usuário tem controle do código, ativar RUM dá field data sem depender do CrUX.

### Bibliotecas

- [`web-vitals`](https://github.com/GoogleChrome/web-vitals) — oficial Google, 1.7KB.
- Next.js: `useReportWebVitals` (app router) ou `reportWebVitals` (pages).
- WordPress: enfileirar `web-vitals` via `wp_enqueue_script`.

### Coleta básica

```js
import {onLCP, onCLS, onINP, onFCP, onTTFB} from 'web-vitals';

function send(metric) {
  navigator.sendBeacon('/api/vitals', JSON.stringify(metric));
}
onLCP(send); onCLS(send); onINP(send); onFCP(send); onTTFB(send);
```

---

## 4. Código e rede

### HTML servido (raw)

```bash
curl -sL https://exemplo.com > page.html

# Inspecionar ordem do <head>
grep -n -E '<(link|script|meta|title)' page.html | head -50

# Tamanho do HTML
wc -c page.html
```

### Headers

```bash
curl -sI https://exemplo.com
# Verificar: Content-Encoding, Cache-Control, CF-Cache-Status, X-Cache, Server-Timing
```

### Assets do hero

Identificar visualmente qual é o LCP element via Lighthouse → `largest-contentful-paint-element`. Depois inspecionar o `<img>`:

```bash
curl -sI https://exemplo.com/hero.jpg
# Tamanho real (Content-Length), MIME type, cache headers
```

### Bundle JS (Next.js / SPA)

```bash
# Build report
ANALYZE=true npm run build  # com @next/bundle-analyzer
# Ou inspecionar treemap no Lighthouse: lh.json → audits.script-treemap-data
```

### Plugins WordPress

```bash
# Via REST API (se exposta)
curl https://exemplo.com/wp-json/

# Via página de plugins (precisa login admin)
# Ou perguntar ao usuário
```

---

## 5. Validação cruzada (CRÍTICO)

Toda métrica deve ser cruzada entre lab × field × código:

| Sintoma | Lab diz | Field diz | Conclusão provável |
|---------|---------|-----------|---------------------|
| LCP ruim | 5s | 1.8s | Lab over-throttles. Field OK. Não priorizar. |
| LCP ruim | 3s | 4.2s | Field pior que lab → cache do CDN/browser está mascarando. Priorizar. |
| INP ruim | (sem dado) | 480ms | TBT lab provavelmente alto também. Investigar third-party. |
| CLS ruim | 0.05 | 0.32 | Lab roda sem cookies/banners; banner injetado no real causa shift. |

---

## Ferramentas externas úteis

- [PageSpeed Insights](https://pagespeed.web.dev/) — lab + field combinados.
- [WebPageTest](https://webpagetest.org) — granular, multi-location, filmstrip.
- [Treo](https://treo.sh/) — RUM hospedado.
- [SpeedCurve](https://speedcurve.com/) — RUM + sintético.
- [Cruxchopper / cruxvis.dev](https://cruxvis.dev/) — explorar CrUX visualmente.
- Chrome DevTools → **Performance Insights** panel (Chrome 102+) — moderno, traduz traces.

---

## Checklist mínimo de coleta antes de auditar

- [ ] Lighthouse mobile rodado 3× (mediana).
- [ ] Lighthouse desktop rodado 3× (mediana).
- [ ] PageSpeed Insights consultado → CrUX p75 anotado.
- [ ] HTML bruto baixado e inspecionado (`<head>`, ordem de scripts).
- [ ] Headers da página + dos 5 maiores assets.
- [ ] LCP element identificado.
- [ ] Top 5 long tasks anotadas (Performance panel).
- [ ] Sources externos listados (third-party-summary).
