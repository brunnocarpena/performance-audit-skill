# Check: CSS

CSS é render-blocking por default. Cada KB de CSS no `<head>` atrasa o FCP e o LCP.

## Auditoria

| Item | Verificar |
|------|-----------|
| Tamanho do CSS bloqueante | `<link rel="stylesheet">` no `<head>` somando > 100KB → red flag |
| CSS não utilizado | Lighthouse `unused-css-rules` |
| CSS crítico inline | Existe `<style>` inline com regras do above-the-fold? |
| Múltiplos stylesheets | Combinar se possível |
| Tailwind | Build com purge/JIT? Ou CDN? (CDN é sempre ruim em prod) |
| Minificação | Comprimido + minified? |
| Preload async pattern | `<link rel="preload" as="style" onload="this.rel='stylesheet'">` |

## Padrão recomendado

### Site simples (landing)
- CSS crítico (hero, tipografia básica) inline no `<head>` — geralmente 5–15KB.
- Resto do CSS via `<link rel="stylesheet">` no fim do `<head>` ou com preload async.

```html
<head>
  <style>/* Critical CSS — gerado por critters/penthouse */</style>
  <link rel="preload" as="style" href="/main.css" onload="this.onload=null;this.rel='stylesheet'">
  <noscript><link rel="stylesheet" href="/main.css"></noscript>
</head>
```

### Site grande (SPA / WordPress)
- CSS crítico difícil de manter manualmente.
- Usar ferramenta automática: **critters** (Next.js), **penthouse**, **critical**.
- Para WP: plugin tipo "Critical CSS" do FlyingPress / WP Rocket / LiteSpeed.

### Tailwind
- **Sempre** rodar build com purge (Tailwind 3+ JIT já faz por default).
- CSS final típico: 5–25KB (vs 3MB do CDN sem purge).
- Safelist classes geradas dinamicamente (estados de form, modals).

## Erros frequentes

| Erro | Consequência |
|------|--------------|
| Tailwind CDN em produção | 3MB+ CSS bloqueante |
| CSS de 500KB no `<head>` | FCP > 3s |
| `@import` em CSS | Cascata sequencial de download |
| Font Awesome 6 inteiro carregado | 80KB+ para usar 5 ícones |
| Bootstrap inteiro carregado | 200KB+ para grid + alguns componentes |
| CSS inline + stylesheet duplicado | Manutenção quebrada, peso |
| `display: none` mass para "esconder" | CSS ainda carrega; melhor não gerar |
| `!important` em todo lugar | Não afeta perf mas indica CSS mal estruturado |
| Animações em `top`/`left` | Não-composited → CLS/INP |

## Animações: composited only

```css
/* RUIM — repaint/reflow */
.bad { transition: top 0.3s, width 0.3s; }

/* BOM — composited */
.good { transition: transform 0.3s, opacity 0.3s; will-change: transform; }
```

Audit: `non-composited-animations`.

## Cross-check obrigatório

1. Lighthouse `unused-css-rules` → identificar páginas com 50%+ CSS não usado.
2. Lighthouse `render-blocking-resources` → identificar stylesheets que poderiam ser async.
3. DevTools → Coverage tab → ver % de CSS usado.
4. Confirmar minificação: arquivo .css sem comentários, sem newlines.
5. Confirmar Brotli/Gzip nos headers (CSS comprime > 70%).

## Riscos de "remover CSS não usado"

- PurgeCSS pode remover classes geradas em runtime (estados, modais, validação de form).
- Sempre manter **safelist** de classes que aparecem só via JS.
- Testar todos os estados (hover, focus, error, loading, modal aberto) antes de deploy.
- Em Elementor/WP: nunca rodar purge agressivo — o builder injeta classes dinamicamente.
