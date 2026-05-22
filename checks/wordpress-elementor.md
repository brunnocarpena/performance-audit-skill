# Check: WordPress / Elementor

WP + Elementor é a combinação mais comum em landing pages brasileiras, e também a fonte mais comum de páginas com Lighthouse mobile < 40.

## Diagnóstico WordPress

| Item | Verificar |
|------|-----------|
| TTFB | > 800ms → hosting/PHP/queries problema |
| Plugins ativos | `wp-cli plugin list` ou painel admin |
| Tema | Tema "multipropósito" pesado (Avada, BeTheme, etc.)? |
| Page caching | WP Rocket / W3TC / LiteSpeed Cache / FlyingPress / hospedagem |
| Object caching | Redis/Memcached? |
| Database | Tabelas com 100k+ linhas em `wp_options` autoload? |
| `wp_head` content | Inspecionar o que cada plugin injeta |
| Heartbeat API | Frequência reduzida? |
| Revisões / lixeira | Limpas? |

## Diagnóstico Elementor (especial)

| Item | Verificar |
|------|-----------|
| Versão | 3.16+ tem Flexbox containers (muito melhor que seções/colunas legadas) |
| Containers Flexbox vs Seções | Flexbox corta DOM em 30–40% |
| Widgets | Quantos? > 100 widgets/página → DOM excessivo, INP ruim |
| Font Awesome 4 | Desabilitar via "Experiments" se não usar ícones FA4 |
| Google Fonts | Desabilitar carregamento global; usar local fonts |
| jQuery | Elementor depende; **não remover** |
| Swiper / Slick | Carrosséis são pesados; lazy + lite mode |
| CSS Print Method | "External File" + cache externo recomendado |
| Custom CSS por widget | Soma rápido; consolidar em CSS global quando possível |
| Animações | Cada animação adiciona JS observer; usar com moderação |

## Configurações recomendadas Elementor (painel)

`Elementor → Settings → Performance`:
- **Improved Asset Loading**: ON
- **Improved CSS Loading**: ON
- **Inline Font Icons (Experimental)**: ON (em vez de Font Awesome global)
- **Optimized Markup**: ON (Elementor 3.18+)
- **Flexbox Container**: ON (em vez de seções)
- **CSS Print Method**: External File
- **Lazy Load Background Images**: ON

`Elementor → Settings → Advanced`:
- **Switch Editor Loader Method**: ON se editor lento

## Plugins recomendados (cache / otimização)

Em ordem de eficácia geral (validar caso a caso):

1. **FlyingPress** — moderno, focado em Core Web Vitals, ~$60/ano.
2. **WP Rocket** — clássico, completo, ~$60/ano.
3. **LiteSpeed Cache** — grátis, melhor com servidor LiteSpeed.
4. **W3 Total Cache** — grátis, complexo, datado.
5. **WP Super Cache** — grátis, simples, só page cache.

**Combos comuns:**
- WP Rocket + Imagify + ShortPixel
- FlyingPress + FlyingCDN (Cloudflare + extras)
- LiteSpeed Cache + QUIC.cloud

## Plugins de imagem

- **ShortPixel** — pago, WebP/AVIF automático.
- **Imagify** — pago, do time WP Rocket.
- **EWWW Image Optimizer** — grátis local + pago cloud.
- **Smush** — grátis básico, lossless.

## Plugins a EVITAR (ou auditar com lupa)

| Plugin | Por quê |
|--------|---------|
| Yoast SEO **+ Rank Math** ativos juntos | Conflito; um só |
| Múltiplos plugins de cache | Conflito; só um |
| Jetpack inteiro (módulos não-essenciais) | Pesa muito; ativar só módulos usados |
| Builders em layers (Elementor + Divi + WPBakery na mesma página) | Caos |
| WP Statistics (rastreia local no DB) | Sobrecarrega DB |
| Newsletter completo se já usa AC/ConvertKit | Redundância |
| Sliders pesados (Revolution Slider, LayerSlider) | Pesadíssimos |

## Cloudflare / CDN

| Setting | Recomendação |
|---------|--------------|
| Auto Minify (HTML/CSS/JS) | **OFF** (quebra muita coisa) |
| Rocket Loader | **OFF** (quebra MUITA coisa, especialmente tracking) |
| Brotli | ON |
| HTTP/3 | ON |
| Polish | Lossless OK; Lossy só se aceitar perda visual |
| Mirage | OFF (legacy mobile feature) |
| Early Hints | ON (Chrome 103+ usa) |
| APO (Automatic Platform Optimization) | $5/mês, vale a pena |

## Red flags WordPress

| Sintoma | Diagnóstico |
|---------|-------------|
| TTFB > 1.5s | Plugins pesados + sem cache + hospedagem ruim |
| Page weight > 4MB | Imagens sem otimização + Elementor com muitos widgets |
| 80+ requests | Plugins injetando CSS/JS próprios |
| `wp-content/plugins/` com 60+ pastas | Plugin bloat |
| `autoload` em `wp_options` > 1MB | Limpar com query |
| Múltiplas versões de jQuery | Conflito de enqueue |

## Auditoria via WP-CLI (se disponível)

```bash
wp plugin list --status=active --fields=name,version,update
wp option get active_plugins | wc -l
wp post list --post_type=revision --format=count
wp transient delete --expired
wp cache flush
```

## Cross-check obrigatório

1. Lighthouse mobile rodado em página real (não admin logado).
2. PageSpeed Insights → field data + lab.
3. GTmetrix waterfall → ordem de carregamento.
4. WP plugin list → identificar plugins desnecessários.
5. `view-source:` da página → contar `<link>` e `<script>` no `<head>`.

## Riscos específicos a alertar SEMPRE

- Cloudflare Rocket Loader **quebra** Elementor sliders + tracking.
- Lazy load de imagem do hero (alguns plugins fazem por default) **quebra LCP**.
- Remover jQuery **quebra forms** do Elementor Pro.
- Combinar CSS agressivamente **quebra** estilos de plugins que dependem de seletores específicos.
- Defer em todos os JS **quebra** plugins que dependem de DOMContentLoaded order.
