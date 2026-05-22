# Check: Imagens

Imagens são, em quase todas as auditorias, o maior peso da página. Quase sempre o LCP element é uma imagem. Quase sempre o CLS é causado por imagens sem dimensão.

## Auditoria de cada imagem

| Item | Verificar |
|------|-----------|
| Formato | WebP/AVIF disponível? Senão, gerar |
| Dimensão | Tamanho servido ≤ tamanho exibido × DPR? |
| Responsive | `srcset` + `sizes` corretos para mobile |
| Lazy | Acima do fold → **NÃO** lazy. Abaixo → `loading="lazy"` |
| Decode | `decoding="async"` em todas (default) |
| Atributos | `width` + `height` sempre (ou `aspect-ratio` CSS) |
| Compressão | qualidade 75–85 para JPEG/WebP, lossless só quando necessário |
| Alt | Presente para SEO + a11y (não relacionado a perf mas inseparável) |
| Hospedagem | CDN com cache? Headers `Cache-Control`? |

## Regras por contexto

### Imagem do hero (LCP)
- **Sem `loading="lazy"`** — quebra LCP.
- `fetchpriority="high"`.
- Preload em `<link rel="preload" as="image">` com `imagesrcset`/`imagesizes`.
- Servida em WebP/AVIF com fallback.
- Tamanho otimizado: < 100KB mobile, < 250KB desktop em geral.

### Imagens abaixo do fold
- `loading="lazy"`.
- `decoding="async"`.
- `width`/`height` ainda obrigatórios (CLS).

### Imagens dentro de carrossel
- Primeiro slide: eager.
- Demais slides: lazy.
- Carrossel não pode lazy o **slide visível inicial** (ele pode ser o LCP).

### Ícones e logos
- Preferir SVG inline para ícones < 5KB.
- Logo: PNG/WebP otimizado, < 20KB.
- Sprite SVG se houver 10+ ícones.

### GIFs
- Converter para `<video>` MP4/WebM com `autoplay muted loop playsinline`.
- Audit Lighthouse `efficient-animated-content`.
- Redução típica: 80–95% do tamanho.

## Padrão correto

```html
<!-- Acima do fold (não-LCP) -->
<img src="/feature-600.webp"
     srcset="/feature-400.webp 400w, /feature-600.webp 600w, /feature-900.webp 900w"
     sizes="(max-width: 768px) 100vw, 600px"
     width="600" height="400"
     decoding="async"
     alt="...">

<!-- Abaixo do fold -->
<img src="/below-600.webp"
     srcset="..."
     sizes="..."
     width="600" height="400"
     loading="lazy"
     decoding="async"
     alt="...">

<!-- Picture com WebP/AVIF + fallback -->
<picture>
  <source type="image/avif" srcset="...">
  <source type="image/webp" srcset="...">
  <img src="/fallback.jpg" width="..." height="..." alt="...">
</picture>
```

## Erros frequentes

| Erro | Consequência | Correção |
|------|--------------|----------|
| Lazy no LCP | LCP > 4s | Remover lazy + preload |
| Sem `width`/`height` | CLS alto | Adicionar atributos ou `aspect-ratio` |
| PNG 2MB no hero | LCP arrasado | Comprimir + WebP |
| `srcset` errado para mobile | Mobile baixando 1800w | Revisar `sizes` |
| Background-image em CSS no hero | LCP não detectado bem, sem fetchpriority | Trocar por `<img>` |
| `object-fit: cover` em img sem altura | CLS | Container com altura fixa |
| Avatar 800×800 exibido 40×40 | Transfer wastage | Servir 80×80 (2× para retina) |

## Cross-check obrigatório

1. Audit `uses-responsive-images` no Lighthouse → lista imagens superdimensionadas.
2. Audit `modern-image-formats` → lista imagens sem WebP/AVIF.
3. Audit `uses-optimized-images` → lista imagens não comprimidas.
4. Audit `unsized-images` → CLS.
5. Audit `offscreen-images` → imagens carregando que não precisariam.

## Pipeline recomendado para gerar imagens

- **Sharp / ImageMagick / Squoosh** — gerar WebP + AVIF + responsive sizes em build.
- **Next.js**: `next/image` (faz tudo automático, mas custa configuração de loader).
- **WordPress**: plugin de WebP (ShortPixel, Imagify, Smush) + servir com `<picture>` ou cabeçalho `Accept`.
- **Cloudflare Polish/Image Resizing** — automático, custo $0–$5/mês.
- **Cloudinary / imgix / Bunny.net** — CDN de imagem com transform on-the-fly.
