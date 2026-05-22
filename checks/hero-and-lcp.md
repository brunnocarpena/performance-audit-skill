# Check: Primeira Dobra e LCP

A primeira dobra é onde 80% das batalhas de performance são ganhas ou perdidas. O LCP element vive aqui em quase 100% dos casos.

## O que verificar

| Item | Verificação | Red flag |
|------|-------------|----------|
| LCP element identificado | Lighthouse → `largest-contentful-paint-element` | Identificar o elemento exato (imagem, headline, vídeo poster) |
| Atributos da imagem LCP | Grep no HTML/template | `loading="lazy"` aplicado → **CRÍTICO**, remover |
| `fetchpriority` no LCP | Conferir `<img>` ou `<picture>` | Faltando → adicionar `fetchpriority="high"` |
| `width` e `height` | Conferir atributos | Faltando → CLS introduzido |
| Preload da imagem LCP | `<link rel="preload" as="image">` no `<head>` | Faltando em hero crítico → adicionar com `imagesrcset` |
| Formato da imagem | `Content-Type` ou extensão | PNG/JPEG quando WebP/AVIF reduziria 30–50% |
| Tamanho da imagem | `Content-Length` ou inspeção visual no PSI | > 200KB em mobile → comprimir / responsive |
| CSS crítico do hero | Procurar `<style>` inline ou stylesheet bloqueante | Stylesheet de 500KB no `<head>` → considerar inline crítico |
| Fonte do headline | Conferir `@font-face` | Sem `font-display: swap` → texto invisível 1–3s |
| JS bloqueante no head | Scripts sem `defer`/`async` antes do hero | GTM/Pixel síncrono → mover ou async |
| Carrossel/slider no hero | É o LCP? | Slider lazy → LCP "vazio" até hidratar |

## Padrão correto para LCP image

```html
<head>
  <link rel="preload" as="image"
        href="/hero-1200.webp"
        imagesrcset="/hero-600.webp 600w, /hero-1200.webp 1200w, /hero-1800.webp 1800w"
        imagesizes="(max-width: 768px) 100vw, 1200px"
        fetchpriority="high">
</head>
<body>
  <picture>
    <source type="image/avif" srcset="/hero-600.avif 600w, /hero-1200.avif 1200w" sizes="(max-width: 768px) 100vw, 1200px">
    <source type="image/webp" srcset="/hero-600.webp 600w, /hero-1200.webp 1200w" sizes="(max-width: 768px) 100vw, 1200px">
    <img src="/hero-1200.jpg"
         srcset="/hero-600.jpg 600w, /hero-1200.jpg 1200w"
         sizes="(max-width: 768px) 100vw, 1200px"
         width="1200" height="600"
         fetchpriority="high"
         decoding="async"
         alt="Descrição do hero (SEO + a11y)">
  </picture>
</body>
```

## Padrão correto para LCP de texto (headline)

```html
<head>
  <link rel="preload" as="font"
        href="/fonts/inter-bold.woff2"
        type="font/woff2"
        crossorigin>
  <style>
    /* Critical: tipografia do hero inline */
    @font-face {
      font-family: 'Inter';
      src: url('/fonts/inter-bold.woff2') format('woff2');
      font-weight: 700;
      font-display: swap;
    }
    h1.hero { font-family: Inter, system-ui, sans-serif; font-weight: 700; }
  </style>
</head>
```

## Erros frequentes

- **Hero atrás de Swiper/Slick lazy** — slider hidrata 2s depois, LCP fica em branco. Solução: primeiro slide estático em HTML.
- **Vídeo de fundo no hero** — `<video autoplay>` é pesado. Usar `poster` otimizado + `preload="metadata"`.
- **Headline atrás de animação JS** — fade-in via `opacity: 0 → 1` no JS atrasa LCP percebido. Solução: pintar visível e animar via CSS.
- **Logo gigante no `<header>` virando LCP** — se o logo for a maior área visível, ele é o LCP. Otimizá-lo como tal.
- **Imagem hero responsiva errada** — mobile baixando versão desktop. `srcset` + `sizes` corretos.

## Cross-check obrigatório

1. Identificar LCP via Lighthouse (mobile e desktop separadamente).
2. Confirmar que **não** tem `loading="lazy"`.
3. Confirmar `fetchpriority="high"`.
4. Confirmar `width`/`height` (CLS).
5. Confirmar `<link rel="preload">` se for imagem crítica.
6. Validar que o preload usa **a mesma URL/srcset** que o `<img>` (senão duplica download).
7. Validar que a fonte do headline tem `font-display: swap` + preload.

## Sinais de sucesso

- LCP < 2.5s em mobile (lab e field).
- LCP element é a imagem/headline esperada (não o footer, não um menu).
- Preload do hero não dispara warning de "unused" no console.
- CLS no hero ≤ 0.05.
