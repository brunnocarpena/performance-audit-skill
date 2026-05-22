# Checks por Domínio

Cada arquivo cobre uma categoria de auditoria. Use o relevante conforme o escopo da sessão.

| Arquivo | Quando consultar |
|---------|------------------|
| [hero-and-lcp.md](hero-and-lcp.md) | Primeira dobra, imagem LCP, headline crítico |
| [images.md](images.md) | Auditoria de todas as imagens (formato, tamanho, lazy, responsive) |
| [fonts.md](fonts.md) | Google Fonts, self-hosted, `font-display`, preload, fallback |
| [css.md](css.md) | Render-blocking, CSS crítico, unused, Tailwind, Elementor CSS |
| [javascript.md](javascript.md) | Defer/async, bundle, hydration Next.js, libs desnecessárias |
| [third-party-tracking.md](third-party-tracking.md) | Pixel, GTM, GA4, Hotmart, AC, Make, chats, consent |
| [wordpress-elementor.md](wordpress-elementor.md) | Plugins, assets globais, Elementor performance, LiteSpeed, Cloudflare |
| [layout-stability.md](layout-stability.md) | CLS, iframes, banners, fonts swap, animações |
| [network-server.md](network-server.md) | Cache, CDN, Brotli, TTFB, preconnect, Early Hints |

## Como combinar

Em uma auditoria completa, percorrer **nessa ordem** (do impacto maior pro menor):

1. Hero + LCP (sempre primeiro)
2. Imagens (próximo maior peso típico)
3. Third-party / Tracking (maior fonte de TBT/INP)
4. JavaScript
5. CSS
6. Fonts
7. Layout stability (CLS)
8. WordPress/Elementor (se aplicável)
9. Network/Server (TTFB, cache, CDN)
