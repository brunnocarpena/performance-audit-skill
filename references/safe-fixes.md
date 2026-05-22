# Correções Seguras vs. Arriscadas

Catálogo de correções comuns de performance, classificadas por **risco de quebrar layout, tracking, conversão ou SEO**.

A skill **nunca** recomenda uma correção sem informar o risco.

---

## Seguras (risco baixo)

Aplicar sem hesitação. Custo de regressão mínimo.

| Correção | Métrica | Por quê é seguro |
|----------|---------|------------------|
| Adicionar `width` e `height` em todas as `<img>` | CLS | Atributos puros, navegador calcula `aspect-ratio` |
| Adicionar `loading="lazy"` em imagens **abaixo do fold** | LCP, transfer | Lazy nativo é estável desde 2019 |
| Adicionar `decoding="async"` em imagens | LCP | Decodificação não-bloqueante, sem efeito visual |
| Adicionar `fetchpriority="high"` na imagem LCP | LCP | Hint, não bloqueia nada |
| Converter PNG/JPEG → WebP **mantendo fallback** | LCP, transfer | `<picture>` com `<source type="image/webp">` |
| Habilitar Brotli/Gzip no servidor | FCP, LCP, TTFB | Apenas compressão, não muda conteúdo |
| Cache-Control de assets versionados (`max-age=31536000`) | LCP repetido | Hash no nome quebra cache quando muda |
| `font-display: swap` em `@font-face` | LCP, CLS | Texto aparece com fallback, depois troca |
| `preconnect` para CDN de fontes/imagens críticas | LCP | Conexão TCP/TLS antecipada |
| Adicionar `defer` em scripts não-críticos (que não estão inline ou no init) | TBT | Mantém ordem de execução |
| Comprimir imagens (mantendo qualidade ≥ 80) | LCP, transfer | Visualmente imperceptível |
| Servir imagens responsivas via `srcset`/`sizes` | LCP mobile | Browser escolhe a versão certa |
| Remover plugins WP comprovadamente não usados | TTFB, TBT | Validar antes que não há dependência |

---

## Médias (risco médio — requer validação)

Aplicar com teste de regressão. Listar explicitamente o que checar depois.

| Correção | Risco | Como validar |
|----------|-------|--------------|
| Preload de imagem LCP | Errar a URL exata (responsive) gera download duplicado | Confirmar `imagesrcset` bate com `<img srcset>` |
| Preload de fonte custom | Errar tipo (woff vs woff2) → warning + duplicação | `type="font/woff2"` + `crossorigin` exatos |
| Trocar Google Fonts CDN por self-hosted | Caching diferente, FOUT pode mudar | Testar em mobile real, comparar CLS |
| Adicionar `async` em script (não `defer`) | Quebra ordem de execução; pode rodar antes de dependência | Validar que script é independente |
| Lazy-load em iframe de embed (YouTube/Vimeo) | Pode quebrar autoplay/eventos | Testar em fluxo real |
| Inline CSS crítico | Aumenta HTML, mata cache do HTML | Só fazer se TTFB-cache compensa |
| Remover CSS não usado (PurgeCSS) | Pode remover classes usadas dinamicamente | Lista de safelist de classes JS-driven |
| Trocar carrossel pesado por estático | Pode mudar a UX percebida | Alinhar com marketing/UX |
| Code-splitting agressivo Next.js | Pode aumentar requests; pode quebrar SSR | Testar hidratação |
| Plugin de cache WP novo (LiteSpeed, WP Rocket) | Pode quebrar checkout, login, member area | Testar todos os fluxos críticos |

---

## Arriscadas (risco alto — só com plano detalhado)

Nunca aplicar sem alinhar com o usuário. Toda recomendação dessa categoria deve incluir o aviso explícito.

| Correção | Risco | Por que cuidado |
|----------|-------|-----------------|
| Mover GTM do `<head>` para `</body>` | **QUEBRA TRACKING** se houver tags com trigger "Page View - Window Loaded" mal configurado, ou consent banner racing tag fire | Validar com tag-assistant + comparar conversões 7d |
| Adicionar `defer` no GTM ou Pixel | **PERDE EVENTOS** disparados antes do `defer` resolver (ex.: scroll rápido) | Pode usar `async` no Pixel + bootstrap dele dentro do dataLayer |
| Server-side tagging (GTM SS) | Migração grande, requer servidor extra, custo | Excelente performance mas projeto à parte |
| Trocar tema/builder do WordPress | Quebra layout inteiro, refaz SEO | Migração com staging + redirect map |
| Migrar WordPress → Next.js / Astro | Custo enorme, refaz tudo | Só com ROI claro |
| Remover jQuery em WP / Elementor | **QUEBRA forms do Elementor Pro, WooCommerce, plugins** | Auditar dependências antes |
| Desativar Heartbeat API do WP | Quebra autosave de admin, integrações | Limitar interval, não desativar |
| Critical CSS inline + remover stylesheet | Pode quebrar segunda página (sem cache) | Manter stylesheet com preload async |
| Cloudflare "Rocket Loader" / Auto Minify JS | **Quebra MUITOS scripts** (Pixel, GTM, sliders) | Quase sempre desabilitar |
| Cloudflare "Mirage" / "Polish Lossy" | Pode degradar qualidade visual de fotos | Validar visualmente |
| Remover plugin antigo de SEO/cache | Pode perder redirects, sitemap, schema | Migrar config primeiro |
| Trocar imagem hero por SVG/WebM | Pode mudar percepção visual | Alinhar com design |
| Adiar `dataLayer.push` por X ms | **Perde eventos** se usuário sai antes | Usar callback de page-load, não setTimeout |

---

## Regras de ouro

1. **Tracking é sagrado.** Antes de mexer em GTM/Pixel/GA4/Hotmart/AC/Make, confirmar com o usuário.
2. **LCP image nunca é lazy.** Sempre `fetchpriority="high"` + sem `loading="lazy"`.
3. **UTMs nunca são dropados.** Redirects/canonical preservam query string.
4. **Layout regressão sempre é testado.** Screenshot diff ou validação visual humana.
5. **Mobile sempre é validado em device real**, não só DevTools throttling.
6. **Antes de remover plugin, audite dependências** (DB de outros plugins, hooks, shortcodes).
7. **Não usar Cloudflare Rocket Loader em sites com tracking.** Quase sempre quebra.
8. **Critical CSS inline só com prova de ROI** — geralmente complica deploy mais do que ajuda.
9. **Score 100 não é a meta.** Core Web Vitals "good" estável é.
10. **Toda correção arriscada → plano de rollback documentado.**
