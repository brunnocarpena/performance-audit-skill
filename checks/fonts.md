# Check: Fontes

Fontes mal configuradas causam três problemas:
1. **FOIT** (texto invisível 1–3s) → LCP ruim.
2. **FOUT** (texto pula da fallback para a custom) → CLS.
3. **Múltiplas requests bloqueantes** → FCP ruim.

## Auditoria de cada fonte

| Item | Verificar |
|------|-----------|
| `font-display` | `swap` (ou `optional` para best perf) |
| Formato | woff2 (>95% browsers); woff só para fallback legado |
| Self-hosted vs Google Fonts | Self-hosted é mais rápido em quase todos os casos |
| Preload | Apenas para fontes do **hero** / first paint |
| Subset | Latin / latin-ext apenas (não carregar cirílico/grego se não usa) |
| Variants (weight, italic) | Carregar só o que é usado |
| `unicode-range` | Útil para fontes multilíngues |

## Padrão correto (self-hosted)

```html
<head>
  <!-- Preload da variante CRÍTICA do hero (só uma!) -->
  <link rel="preload" as="font"
        href="/fonts/inter-700.woff2"
        type="font/woff2"
        crossorigin>

  <style>
    @font-face {
      font-family: 'Inter';
      src: url('/fonts/inter-400.woff2') format('woff2');
      font-weight: 400;
      font-display: swap;
      unicode-range: U+0000-00FF, U+0131, U+0152-0153;
    }
    @font-face {
      font-family: 'Inter';
      src: url('/fonts/inter-700.woff2') format('woff2');
      font-weight: 700;
      font-display: swap;
    }
  </style>
</head>
```

## Padrão correto (Google Fonts, quando inevitável)

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link rel="preload" as="style"
      href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap">
<link rel="stylesheet"
      href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap">
```

> Google Fonts **sempre** com `display=swap` na query.

## Estratégia de fallback (anti-CLS)

CSS fallback metric overrides reduzem CLS do swap drasticamente:

```css
@font-face {
  font-family: 'Inter Fallback';
  src: local('Arial');
  ascent-override: 90.20%;
  descent-override: 22.48%;
  line-gap-override: 0%;
  size-adjust: 107.40%;
}

body {
  font-family: 'Inter', 'Inter Fallback', system-ui, sans-serif;
}
```

Next.js gera isso automaticamente via `next/font`.

## Erros frequentes

| Erro | Consequência |
|------|--------------|
| `font-display: block` (default) | FOIT 3s → LCP ruim |
| 6+ variantes carregadas (regular, bold, italic, light, black...) | Peso desnecessário |
| Preload de fonte não-crítica | Atrasa LCP image |
| Preload errando tipo (`font/woff` em vez de `font/woff2`) | Warning + duplicação |
| Falta `crossorigin` no preload de fonte | Refetch (Chrome) |
| Google Fonts sem preconnect | TLS + DNS bloqueia render |
| Self-hosted sem `Cache-Control` longo | Re-download a cada visita |
| Font Awesome 4 carregando o set inteiro | 200KB+ desnecessário |

## Cross-check obrigatório

1. Audit `font-display` no Lighthouse — todas as `@font-face` com `swap` ou `optional`.
2. Audit `preload-fonts` — fontes críticas têm preload.
3. Audit `uses-text-compression` — woff2 (já comprimido).
4. Conferir DevTools → Network → Font → ver quantas requests.
5. Confirmar que preload **não dispara warning** "preloaded but not used within X seconds".

## Decisão: Google Fonts vs Self-hosted?

| Caso | Recomendação |
|------|--------------|
| Landing page única, fonte muito usada | **Self-hosted** (mais rápido) |
| Site grande com fontes variadas | **Self-hosted** (controle de cache) |
| Brand 100% confiando em fonte do Google que não vai mudar | Self-hosted; baixe os arquivos uma vez |
| Protótipo / MVP | Google Fonts (rápido de configurar) |
| Conformidade com GDPR | **Self-hosted** (Google Fonts já foi processado por LGPD/GDPR em alguns países) |
