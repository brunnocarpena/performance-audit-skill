# Rubrica de Severidade — Performance Audit

Critérios para classificar problemas de performance encontrados na auditoria.

A severidade considera **impacto na métrica × impacto no negócio × tamanho da base afetada**.

---

## Crítica

**Impacto:** Page falha Core Web Vitals no field data (CrUX p75 fora do "good"), OU quebra fluxo de conversão por lentidão/instabilidade, OU faz a página perder ranking ativamente.

**Ação:** **Bloqueio obrigatório** de deploy/ranqueamento até correção.

### Exemplos

| Contexto | Problema |
|----------|----------|
| Hero | Imagem LCP de 2MB+ sem otimização → LCP 6s+ no mobile |
| Hero | `loading="lazy"` aplicado na imagem LCP → LCP > 4s |
| JS | Bundle de 1.5MB+ síncrono no `<head>` → TBT > 1s, INP > 500ms |
| Fontes | Google Fonts bloqueando render sem `font-display`, FOIT > 3s |
| CLS | Banner cookie/oferta injetado após render move o hero inteiro → CLS > 0.4 |
| Tracking | Página rápida mas Pixel/GA4 não dispara → conversão atribuída zerada |
| Mobile | Score mobile < 40 com tráfego majoritariamente mobile |
| Plugins WP | 80+ plugins ativos somando 3s+ de TTFB |
| Elementor | Página com 200+ widgets → DOM > 3000 nodes, INP > 800ms |

---

## Alta

**Impacto:** Lighthouse falha uma das CWV (lab) ou field está em "needs improvement", há regressão visível na primeira dobra, OU usuário em mobile médio espera > 3s para interagir.

**Ação:** **Bloqueio recomendado**. Liberar apenas com plano de correção e prazo.

### Exemplos

| Contexto | Problema |
|----------|----------|
| Imagens | Imagens servidas em PNG/JPEG sem WebP/AVIF, somando MB do peso da página |
| CSS | CSS render-blocking somando 500KB+ no `<head>` |
| Fontes | 4+ variantes de fonte carregadas (regular, bold, italic, etc.) sem necessidade |
| Tracking | GTM no `<head>` síncrono — bloqueia render por 200–500ms |
| Carrossel | Slider com 8+ imagens carregando tudo (sem lazy abaixo da primeira) |
| Vídeo | YouTube iframe carregando completo no load (não lite-embed) |
| Cache | Sem cache de página em WordPress, TTFB > 1.5s |
| Next.js | Componente client-side hidratando 200KB+ de JS no hero |

---

## Média

**Impacto:** Otimizações importantes mas que não derrubam métricas para "poor". Páginas com vários "média" somam para "alta".

**Ação:** Corrigir antes do deploy se possível. Aceitável com ressalvas.

### Exemplos

| Contexto | Problema |
|----------|----------|
| Imagens | Imagens com dimensões nativas maiores que o display (1200px exibida em 600px) |
| CSS | Tailwind CDN em produção (poderia ser build com purge) |
| JS | Lib jQuery carregada mas usada em < 5 trechos |
| Preconnect | Faltando preconnect para CDN de imagens / fontes externas |
| Compressão | Resposta sem Brotli (só Gzip) |
| Defer | Scripts não-críticos sem `defer`/`async` |
| Elementor | Ícones Font Awesome 4 carregados (poderia ser FA 5 + ícones específicos) |

---

## Baixa

**Impacto:** Otimizações cosméticas ou de longo prazo. Não afetam ranking nem percepção do usuário no curto prazo.

**Ação:** Corrigir quando conveniente. Não bloqueia deploy.

### Exemplos

| Contexto | Problema |
|----------|----------|
| HTML | `<head>` com 2–3 comments verbose desnecessários |
| Imagens | SVG inline poderia ser sprite (microoptimização) |
| Cache | `Cache-Control: max-age=86400` poderia ser `31536000` em assets versionados |
| DNS | `dns-prefetch` poderia ser `preconnect` em domínio crítico |
| Console | Warnings de console (não erros) |

---

## Como atribuir

Decidir nessa ordem:

1. **A métrica afetada está em "poor" no field?** → crítica.
2. **A métrica afetada está em "poor" no lab OU "needs" no field?** → alta.
3. **Há "wasted bytes/ms" no Lighthouse > 300ms ou > 200KB?** → média.
4. **É melhoria sem impacto mensurável imediato?** → baixa.

Combinar com **risco da correção** (em [safe-fixes.md](safe-fixes.md)) para definir prioridade P0/P1/P2 — não só severidade.
