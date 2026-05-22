# Check: Rede e Servidor

Performance que importa começa **antes** do primeiro byte. TTFB ruim trava todas as métricas downstream.

## Auditoria

| Item | Verificar |
|------|-----------|
| TTFB | `curl -w "%{time_starttransfer}"` ou Lighthouse `server-response-time` |
| Compressão | Header `Content-Encoding: br` (Brotli) ou `gzip` |
| HTTP version | HTTP/2 mínimo; HTTP/3 ideal |
| Cache de página | `Cache-Control` no HTML (cuidado com login/personalizado) |
| Cache de assets | `Cache-Control: public, max-age=31536000, immutable` em assets hashados |
| CDN | Header `CF-Cache-Status` / `X-Cache` / `Server-Timing` |
| Preconnect | Para origens críticas externas |
| DNS-prefetch | Para origens secundárias |
| Early Hints (103) | Suportado pelo Chrome 103+ |
| Service Worker | Útil para sites repetidamente acessados |

## Headers ideais

### HTML dinâmico (WP, Next ISR)
```
Cache-Control: public, max-age=0, s-maxage=300, stale-while-revalidate=86400
Content-Encoding: br
Server-Timing: cache;desc="HIT"
```

### Assets versionados (CSS, JS, fonts)
```
Cache-Control: public, max-age=31536000, immutable
Content-Encoding: br
```

### Imagens
```
Cache-Control: public, max-age=2592000
```

## Preconnect / DNS-prefetch

```html
<head>
  <!-- Preconnect: estabelece TCP + TLS antes do request -->
  <link rel="preconnect" href="https://cdn.example.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link rel="preconnect" href="https://www.googletagmanager.com">

  <!-- DNS-prefetch: só resolve DNS, mais barato -->
  <link rel="dns-prefetch" href="https://connect.facebook.net">
  <link rel="dns-prefetch" href="https://stats.g.doubleclick.net">
</head>
```

**Regras:**
- Máximo 4–6 preconnects (cada um custa recursos).
- Preconnect só para origens **usadas na primeira dobra**.
- DNS-prefetch para origens **prováveis** mas não certas.

## CDN — opções comuns

| CDN | Pontos fortes | Pontos fracos |
|-----|---------------|---------------|
| **Cloudflare** | Free tier generoso, Workers, R2 | DNS na Cloudflare obrigatório no free |
| **BunnyCDN** | Barato, performance excelente | UI simples |
| **Vercel/Netlify Edge** | Integrado com framework | Lock-in |
| **AWS CloudFront** | Robusto, custo previsível | Configuração complexa |
| **Fastly** | Performance topo, VCL flexível | Caro |

## Brotli vs Gzip

- **Brotli** comprime ~20% melhor que Gzip em texto.
- Todos os browsers modernos suportam (`Accept-Encoding: br`).
- Servidor deve negociar (`Vary: Accept-Encoding`).
- Nginx: `brotli on; brotli_types text/css text/javascript application/json ...`
- Apache: `mod_brotli`.

## TTFB diagnóstico

```bash
curl -o /dev/null -w "DNS: %{time_namelookup}s\nTLS: %{time_appconnect}s\nTTFB: %{time_starttransfer}s\nTotal: %{time_total}s\n" -s https://exemplo.com
```

**Targets:**
- DNS < 50ms (CDN/anycast).
- TLS < 200ms.
- TTFB total < 800ms (good), < 200ms (ótimo).

**Causas comuns de TTFB alto:**
- WordPress sem page cache.
- Queries lentas no DB.
- PHP sem OPcache.
- SSR pesado (Next sem ISR).
- Origem geográfica longe do usuário (sem CDN edge).
- Plugins WP sobrecarregando init.

## Service Worker

Para sites que o usuário visita repetidamente:
- Cache de assets first → repeat-load instantâneo.
- Offline fallback.
- Workbox library: configuração simples.
- **Cuidado:** SW persiste; bug pode "travar" site no cache de uma versão antiga. Sempre versionar + skipWaiting.

## Early Hints (HTTP 103)

```
HTTP/2 103 Early Hints
Link: </style.css>; rel=preload; as=style
Link: </main.js>; rel=preload; as=script
```

Servidor envia hints **antes** do HTML final. Cloudflare suporta nativo, Cloudfront via funções.

## Cross-check obrigatório

1. `curl -sI https://...` → conferir headers.
2. Lighthouse `server-response-time` → TTFB.
3. Lighthouse `uses-text-compression` → Brotli/Gzip.
4. Lighthouse `uses-long-cache-ttl` → cache de assets.
5. WebPageTest waterfall → ver onde o tempo está indo.

## Red flags

| Sintoma | Causa provável |
|---------|----------------|
| TTFB 2s+ | WP sem cache + hospedagem ruim |
| `Cache-Control: no-cache` em CSS/JS | Cache desperdiçado |
| `Set-Cookie` em assets estáticos | Cookie size em cada asset |
| HTTP/1.1 | Servidor desatualizado |
| Sem Brotli | -20% de eficiência de compressão |
| 10+ preconnects | Recursos desperdiçados |
| Preconnect para origens não usadas | Warning + waste |
