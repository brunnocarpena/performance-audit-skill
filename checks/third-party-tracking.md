# Check: Terceiros e Tracking

**Esta é a check mais sensível.** Mexer em tracking quebra atribuição, conversão e relatórios de marketing. Toda recomendação aqui exige confirmação explícita do usuário antes de aplicar.

## Princípio guia

```
PERFORMANCE NUNCA JUSTIFICA QUEBRAR TRACKING.
```

Score 100 + Pixel/GA4 morto = página inútil para o negócio. A meta é tracking funcional **e** performance boa, não uma ou outra.

## Inventário típico em landing/captura

| Ferramenta | Função | Risco de quebrar |
|------------|--------|-------------------|
| Google Tag Manager | Container | Alto — orquestra todas as outras tags |
| Meta Pixel | Conversão FB/IG ads | Alto |
| Google Analytics 4 | Analytics | Médio |
| Google Ads tag | Conversão Google Ads | Alto |
| Hotmart pixel | Conversão de afiliado | Alto (atribuição) |
| ActiveCampaign tracking | Lead scoring + automation | Alto |
| Make (Integromat) webhooks | Automação backend | Médio (não bloqueia render) |
| LinkedIn Insight Tag | Conversão LI ads | Médio |
| TikTok Pixel | Conversão TT ads | Médio |
| Hotjar / Clarity / FullStory | Heatmap/session replay | Alto custo de perf, baixo risco se quebrar |
| Intercom / Drift / Crisp / Tawk | Chat | Médio |
| Cookie consent (Cookiebot, OneTrust) | LGPD/GDPR | Alto — orquestra disparo de tags |

## Padrão recomendado para cada um

### Google Tag Manager
- Snippet padrão no `<head>` (síncrono, mas pequeno).
- Tags **dentro** do GTM disparadas em "Window Loaded" quando não-críticas.
- Tags críticas (Pixel PageView, GA4 PageView) em "All Pages" / "DOM Ready".
- Server-side tagging (sGTM) se orçamento permite — melhor performance.
- **Nunca** colocar GTM com `defer` ou `async` sem consultar o time de mídia.

### Meta Pixel
```html
<!-- Base code no <head> -->
<script>
  !function(f,b,e,v,n,t,s){...}('https://connect.facebook.net/en_US/fbevents.js');
  fbq('init', 'PIXEL_ID');
  fbq('track', 'PageView');
</script>
```
- Aceitável `async` se carregado via GTM com trigger "Page View".
- **Sempre** preconnect `connect.facebook.net`.

### GA4
- Via GTM ou direto via `gtag.js`.
- `gtag.js` já é assíncrono por design.
- Preconnect `www.googletagmanager.com`.

### Hotmart / ActiveCampaign / etc.
- Idem padrão de tag — `async` ou via GTM.
- **Confirmar com usuário** antes de mover ou adiar.

### Chat widgets (Crisp, Intercom)
- Lazy-load **após** primeira interação ou scroll.
- Estratégia: container vazio + carregar script no `requestIdleCallback` ou no primeiro click.

```js
// Lazy chat
let chatLoaded = false;
const loadChat = () => {
  if (chatLoaded) return; chatLoaded = true;
  const s = document.createElement('script');
  s.src = '/chat.js'; s.async = true;
  document.body.appendChild(s);
};
['scroll','click','mousemove','touchstart'].forEach(e =>
  window.addEventListener(e, loadChat, {once:true, passive:true})
);
setTimeout(loadChat, 8000); // fallback
```

### Cookie consent banners
- Causam CLS se aparecem após render → reservar espaço no top/bottom.
- Causam INP alto se bloqueiam tags síncronas.
- Validar que consent → tag fire timing está OK.

## Preconnect obrigatório

Para qualquer terceiro carregado no início:

```html
<link rel="preconnect" href="https://www.googletagmanager.com">
<link rel="preconnect" href="https://www.google-analytics.com">
<link rel="preconnect" href="https://connect.facebook.net">
<link rel="dns-prefetch" href="https://stats.g.doubleclick.net">
```

## Red flags

| Sintoma | Provável causa |
|---------|----------------|
| TBT > 800ms em mobile | GTM síncrono carregando 20+ tags no init |
| INP > 500ms | Tags com `event handler` síncrono pesado |
| LCP atrasado mesmo sem imagem grande | Script de A/B test bloqueando render |
| Múltiplas chamadas a `connect.facebook.net` | Pixel duplicado |
| `dataLayer.push` chamado mas tag não dispara | Trigger errado no GTM |
| Score 100 mas relatório de Ads cai | Tag movida sem trigger correto |

## Antes de qualquer otimização em tracking

Checklist obrigatório:

- [ ] **Confirmar com usuário** que pode mexer.
- [ ] Documentar configuração atual (screenshot do GTM, lista de tags ativas).
- [ ] Identificar tags críticas (PageView, ViewContent, Lead, Purchase) vs nice-to-have.
- [ ] Testar em ambiente de preview (GTM Preview Mode, Facebook Pixel Helper).
- [ ] Comparar eventos disparados ANTES e DEPOIS — número exato.
- [ ] Validar atribuição em painel de mídia 24–72h pós-deploy.
- [ ] Plano de rollback em < 5 min se algo quebrar.

## Otimizações seguras (sem mexer em config de tag)

1. **Preconnect** para domínios críticos.
2. **DNS-prefetch** para domínios secundários.
3. **Lazy chat widget** com fallback timeout.
4. **Migrar para sGTM** (projeto separado, sem risco no front).
5. **Partytown** para mover GTM/Pixel pra web worker (testar bem).
6. **Reduzir número de tags** removendo as não usadas (auditoria de GTM).

## Cross-check obrigatório

1. Lighthouse `third-party-summary` → ranking de blocking time por origin.
2. Lighthouse `bootup-time` → JS por script.
3. DevTools Network → cascata; ver se third-party está bloqueando LCP.
4. Tag Assistant / Pixel Helper → confirmar disparo correto.
5. GA4 DebugView → confirmar eventos.

## UTMs e parâmetros — NUNCA dropar

- Redirects (301/302) precisam preservar query string.
- Canonical não pode apontar para versão sem UTM.
- AMP/PWA não pode reescrever URL sem UTMs.
- SPA precisa propagar UTM para chamadas internas se houver atribuição multi-step.
