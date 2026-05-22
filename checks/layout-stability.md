# Check: Estabilidade Visual (CLS)

CLS é a métrica mais frustrante para o usuário: o conteúdo "pula" enquanto ele lê ou clica. Também é a mais fácil de corrigir quando identificada cedo.

## Princípio

```
TODO ELEMENTO QUE OCUPARÁ ESPAÇO PRECISA TER O ESPAÇO RESERVADO ANTES DELE CARREGAR.
```

## Causas comuns

| Elemento | Como reservar espaço |
|----------|----------------------|
| `<img>` | `width` + `height` (browser calcula `aspect-ratio`) |
| `<img>` responsiva | `width` + `height` em proporção, CSS `width: 100%; height: auto` |
| `<iframe>` (vídeo) | Container com `aspect-ratio: 16/9` |
| Vídeo embed | Container fixo, lazy load só do iframe |
| Banner/cookie consent | Posição absoluta no topo, com `min-height` reservado |
| Ad slots | Container com altura fixa pre-render |
| Fontes custom | `font-display: swap` + size-adjust fallback |
| Botões que aparecem após hydration | Renderizar placeholder visualmente idêntico |
| Conteúdo dinâmico (avaliações, contadores) | Skeleton com mesma altura |
| Modais / drawers | `position: fixed`, não empurram layout |
| Acordeões / FAQ | `max-height` transitions OK; CSS Grid OK |

## Padrões corretos

### Imagem responsiva sem CLS
```html
<img src="..." srcset="..." sizes="..."
     width="1200" height="600"
     style="width: 100%; height: auto;"
     alt="...">
```

### Iframe responsivo (YouTube/Vimeo)
```html
<div style="aspect-ratio: 16/9; width: 100%;">
  <iframe src="..." style="width:100%; height:100%; border:0;" loading="lazy"
          allow="autoplay; encrypted-media" allowfullscreen></iframe>
</div>
```

### Lite-embed para YouTube (recomendado)
[lite-youtube-embed](https://github.com/paulirish/lite-youtube-embed) — 3KB, carrega iframe real só no click.

### Cookie banner
```css
.cookie-banner {
  position: fixed;
  bottom: 0; left: 0; right: 0;
  /* não empurra conteúdo */
}
```

### Fonts (FOUT sem CLS)
```css
@font-face {
  font-family: 'Inter Fallback';
  src: local('Arial');
  size-adjust: 107.40%;
  ascent-override: 90.20%;
  descent-override: 22.48%;
}
body { font-family: 'Inter', 'Inter Fallback', sans-serif; }
```

## Animações sem CLS

```css
/* RUIM — afeta layout */
.bad { transition: width 0.3s, height 0.3s, margin 0.3s; }

/* BOM — composited, sem CLS */
.good { transition: transform 0.3s, opacity 0.3s; }
```

CSS `transform: translateY()` em vez de mexer em `top` / `margin`.

## Animações de entrada (hero / cards)

```css
/* OK: animar opacity + transform */
.fade-in {
  opacity: 0;
  transform: translateY(20px);
  transition: opacity 0.4s ease, transform 0.4s ease;
}
.fade-in.visible { opacity: 1; transform: translateY(0); }
```

Para **scroll-triggered**, usar `IntersectionObserver`, não scroll listener.

## Erros frequentes

| Erro | Consequência |
|------|--------------|
| Imagem sem `width`/`height` | CLS por cada imagem |
| Iframe YouTube full sem aspect-ratio | Shift gigante quando carrega |
| Banner injetado no `<body>` topo | Empurra hero pra baixo → CLS > 0.2 |
| Animação `top: -20px → 0` | Layout shift |
| `display: none → block` em conteúdo crítico | Shift |
| Skeleton com altura errada | Shift quando real carrega |
| Botão "Ver mais" que expande conteúdo no hero | OK se previsto (interação do usuário não conta para CLS) |
| Lazy load com placeholder vazio sem altura | Shift |

## CLS por contexto (mobile vs desktop)

- Mobile geralmente tem CLS pior — menos viewport, banner ocupa proporcionalmente mais.
- Desktop com newsletters/pop-ups inline pode ter CLS surpreendente.
- Field data (CrUX) reporta os dois separadamente.

## Cross-check obrigatório

1. Lighthouse `layout-shift-elements` → lista elementos que shiftaram.
2. Lighthouse `unsized-images` → imagens sem dimensão.
3. Lighthouse `non-composited-animations` → animações ruins.
4. DevTools → Performance → Layout Shift track.
5. CrUX → CLS p75 separado mobile/desktop.

## Importante: shift de interação NÃO conta

CLS oficial só conta shifts **inesperados** (sem interação do usuário nos últimos 500ms). Então:
- Expandir um acordeão por click → não conta.
- Modal abrindo após click → não conta.
- Cookie banner aparecendo sozinho 2s após load → **conta**.
- Ad slot carregando após hero → **conta**.
