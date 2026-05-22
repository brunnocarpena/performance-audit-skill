---
name: performance-audit
description: Use sempre que precisar auditar performance de uma página (landing, WordPress/Elementor, HTML/Tailwind, Next.js, página de captura) sob a ótica do Google PageSpeed Insights / Lighthouse, OU quando estiver desenvolvendo uma página nova e quiser garantir boa pontuação desde o início. TRIGGER quando o agente menciona "PageSpeed", "Lighthouse", "Core Web Vitals", "LCP", "CLS", "INP", "TBT", "performance", "página lenta", "score baixo", "otimizar página"; quando a sessão toca arquivos de hero, imagens, fontes, CSS crítico, GTM/Pixel/Analytics, plugins de cache, Elementor; antes de qualquer deploy de página pública. SKIP em mudanças puramente internas (API, jobs, schema) sem impacto na renderização do front.
argument-hint: [url-ou-escopo?] [--mode=audit|build] [--target=mobile|desktop|both]
allowed-tools: Read, Grep, Glob, Bash(curl:*), Bash(git diff:*), Bash(git status:*), Bash(ls:*), Bash(find:*), Bash(wc:*), WebFetch, Task
---

# Performance Audit

Auditoria de performance de páginas sob a ótica do **Google PageSpeed Insights / Lighthouse**, sem sacrificar layout, rastreamento, UTMs, SEO ou conversão.

**Esta skill não é uma otimização cega.** É uma auditoria com critério, que separa o que o Lighthouse mede de o que o usuário sente, e prioriza correções que entregam impacto real sem quebrar a página.

## Princípio central

```
PERFORMANCE NÃO É SCORE. É ENTREGAR A PRIMEIRA DOBRA RÁPIDO,
MANTER ESTABILIDADE VISUAL E NÃO QUEBRAR TRACKING NEM CONVERSÃO.
```

Um score 100 com formulário quebrado, Pixel não disparando ou UTM perdido **é uma regressão**, não uma vitória. Toda recomendação desta skill deve preservar:

1. **Layout visual** — sem CLS introduzido, sem quebra de grid/componentes.
2. **Tracking** — Meta Pixel, GTM, GA4, Hotmart, ActiveCampaign, Make, conversões.
3. **UTMs e parâmetros de URL** — nunca dropar query string em redirects/canonical.
4. **Funcionalidade de formulários** — campos, validação, action, integrações.
5. **SEO** — meta tags, structured data, hreflang, canonical, robots.
6. **Acessibilidade básica** — alt, labels, contraste, foco.

Se uma "otimização" coloca qualquer um destes em risco, **não recomende sem alerta explícito**.

## Modos de operação

A skill opera em dois modos. Identifique o modo no início:

### Modo `audit` — auditoria de página existente

Inputs aceitos:
- URL pública.
- Print/JSON do PageSpeed Insights.
- Relatório Lighthouse (HTML/JSON).
- Código-fonte (HTML, template Elementor, componente Next.js).
- Trecho específico (hero, head, footer).

Output: diagnóstico priorizado + plano de correção (ver [references/report-format.md](references/report-format.md)).

### Modo `build` — desenvolvendo página nova

Aplicar boas práticas **antes** do código existir. Revisar HTML/CSS/JS no commit. Validar hero, imagens, fontes, scripts e tracking conforme as checks em [checks/](checks/).

Output: checklist pré-deploy + apontamentos de risco no código atual.

## Workflow

### 1. Identificar escopo e modo

- Se houver URL ou relatório Lighthouse → `audit`.
- Se estiver editando arquivos de página/componente sem relatório → `build`.
- Se ambíguo, perguntar **uma vez**: "Você quer auditar página existente ou revisar código antes de deploy?"

### 2. Coletar evidência

**Pré-check obrigatório — API key do PageSpeed Insights**

Antes de qualquer coleta, verificar se `$PSI_API_KEY` está configurada:

```bash
if [ -z "$PSI_API_KEY" ]; then
  echo "PSI_API_KEY ausente"
fi
```

**Se a variável NÃO estiver setada, pausar e avisar o usuário com este texto** (adaptar tom à conversa, manter o conteúdo):

> ⚠️ Você ainda não configurou a chave da API do PageSpeed Insights (`$PSI_API_KEY`). Sem ela eu fico limitado a ~1 query/dia keyed por IP — e bate quota rápido, me forçando a fazer auditoria só estática (sem score, sem field data CrUX, sem audits do Lighthouse).
>
> **Com a chave configurada, eu consigo ser muito mais assertivo:** trago Performance Score real, LCP/CLS/INP/TBT/FCP medidos, opportunities priorizadas, breakdown do main thread, scripts pesados identificados, e dados de campo (CrUX) de usuários reais nos últimos 28 dias.
>
> Setup (2 min):
> 1. https://console.cloud.google.com/apis/credentials → criar API key
> 2. Habilitar **PageSpeed Insights API** e **Chrome UX Report API** em APIs & Services → Library
> 3. Adicionar no `~/.zshrc`:
>    ```bash
>    export PSI_API_KEY="AIza..."
>    export CRUX_API_KEY="$PSI_API_KEY"
>    ```
> 4. `source ~/.zshrc` ou abrir terminal novo
>
> Quer configurar agora, ou prefere que eu siga com auditoria estática limitada?

Se o usuário optar por configurar: aguardar e validar com `curl` de teste. Se optar por seguir sem: marcar no relatório final **"Auditoria parcial — sem dados PSI/CrUX"** e basear conclusões apenas em análise estática (HTML/headers/assets).

**Coleta efetiva:**

| Fonte | O que extrair |
|-------|---------------|
| URL pública | Rodar `curl -sI` para headers, baixar HTML, inspecionar `<head>`, ordem dos scripts, imagens do hero |
| Relatório Lighthouse | Métricas (LCP/CLS/INP/TBT/FCP/SI), top opportunities, diagnostics, treemap JS |
| Código | Hero, ordem de `<link>`/`<script>`, atributos de imagem, fontes, CSS crítico, plugins (WP), bundle (Next) |
| PageSpeed Insights | Lab data **e** field data (CrUX) separados, mobile **e** desktop separados |

Para WebFetch / curl em URL pública, consultar [references/data-collection.md](references/data-collection.md). Para mecânica de PSI/Lighthouse/CrUX (lab×field, pesos do score, INP, soft-nav, throttling, PSI API v5), consultar [references/psi-lighthouse-internals.md](references/psi-lighthouse-internals.md).

### 3. Classificar por categoria

Toda análise é organizada em **12 categorias** (ver [checks/](checks/)):

1. Resumo executivo
2. Métricas PageSpeed (LCP, CLS, INP/TBT, FCP, SI)
3. Primeira dobra (hero, headline, CTA, CSS crítico)
4. Imagens
5. Fontes
6. CSS
7. JavaScript
8. Terceiros e tracking
9. WordPress / Elementor
10. Estabilidade visual (CLS)
11. Rede e servidor
12. Plano de ação

### 4. Atribuir impacto e severidade

Cada problema recebe:
- **Métrica afetada** (LCP, CLS, INP, TBT, FCP, SI, TTFB).
- **Severidade** (crítica / alta / média / baixa) — ver [references/severity-rubric.md](references/severity-rubric.md).
- **Prioridade** (P0 / P1 / P2) — baseada em impacto × custo de correção.
- **Risco da correção** (quebra layout? quebra tracking? requer regressão visual?).

### 5. Recomendar correção segura

Para cada problema:
- O que mudar exatamente (arquivo / seletor / atributo).
- Por que isso afeta a métrica X.
- O que NÃO mexer junto.
- Como testar depois (Lighthouse run, WebPageTest, DevTools Performance, regressão visual).

Lista de correções seguras vs. arriscadas: [references/safe-fixes.md](references/safe-fixes.md).

### 6. Emitir relatório

Template obrigatório: [references/report-format.md](references/report-format.md).

### 7. Decisão pré-deploy

- Algum problema **crítico** não resolvido → **bloqueia deploy**.
- Múltiplos **altos** em hero/LCP → **bloqueia** até corrigir os top 3.
- Apenas **médios/baixos** → **aprovado com ressalvas**.
- Tudo verde + Core Web Vitals dentro do threshold → **aprovado**.

## Métricas e thresholds (referência rápida)

Valores 2026 do Core Web Vitals (alinhados com Google CrUX). Sempre validar contra [web.dev/vitals](https://web.dev/articles/vitals) se houver dúvida.

| Métrica | Bom | Precisa melhorar | Ruim |
|---------|-----|------------------|------|
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | 2.5s – 4.0s | > 4.0s |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | 0.1 – 0.25 | > 0.25 |
| **INP** (Interaction to Next Paint) | ≤ 200ms | 200ms – 500ms | > 500ms |
| **TBT** (Total Blocking Time, lab) | ≤ 200ms | 200ms – 600ms | > 600ms |
| **FCP** (First Contentful Paint) | ≤ 1.8s | 1.8s – 3.0s | > 3.0s |
| **Speed Index** | ≤ 3.4s | 3.4s – 5.8s | > 5.8s |
| **TTFB** | ≤ 800ms | 800ms – 1800ms | > 1800ms |

Detalhes em [references/metrics-glossary.md](references/metrics-glossary.md).

## Lab Data vs. Field Data — diferenciar SEMPRE

- **Lab data (Lighthouse)** — simulação em throttling fixo (Moto G4 / 4G lento). Útil para diagnóstico.
- **Field data (CrUX)** — dados reais de usuários Chrome nos últimos 28 dias. Útil para priorização real.
- **Discrepância comum:** Lighthouse mostra LCP 4s mas CrUX mostra 2.1s — usuário real está bem, score do Lighthouse não. **Decida com base no field, valide com lab.**
- Mobile e desktop **têm rankings separados no Google**. Sempre avaliar ambos.

## Red Flags — STOP imediatamente

Se você se pegar pensando ou recomendando:

- "Adiar todos os scripts com `defer`" — sem mapear dependências de tracking/inline.
- "Lazy-load em todas as imagens" — incluindo a imagem do LCP.
- "Remover o Pixel/GTM porque pesa muito" — sem alinhar com marketing.
- "Trocar Elementor por código nativo" — sem entender o ciclo de edição.
- "Comprimir a hero pra 30KB" — perdendo qualidade visual da headline.
- "Remover CSS não usado automaticamente" — pode quebrar estados (hover, modal, error).
- "Score 100 no Lighthouse é a meta" — meta é Core Web Vitals "good" no field data.
- "Migrar pra Next.js / framework X resolve" — sem entender o problema real.

→ STOP. Volte para a etapa 4 (impacto × risco) antes de recomendar.

## Excuse | Reality

| Desculpa | Realidade |
|----------|-----------|
| "Lighthouse só roda em mobile lento, não é real" | É o ranking que o Google usa. Field data corrobora. |
| "Score 80 já tá bom" | Core Web Vitals é binário (passa/não passa). 80 pode falhar no LCP. |
| "Otimizei tudo e o score caiu" | Você provavelmente quebrou o tracking inline ou o LCP. Reaudite. |
| "O cliente quer 100" | 100 é frágil. Meta é "all green CWV" estável. Eduque o cliente. |
| "Vou usar plugin X que faz tudo" | Plugins fazem 60% e quebram 30%. Audite o resultado, não o plugin. |
| "Mobile não importa, audiência é desktop" | Mobile-first index do Google ranqueia pelo mobile. |
| "Removi as fontes custom e ficou rápido" | Conversão caiu junto. Otimize a fonte, não remova. |

## Checks por domínio

Use o arquivo da check apropriada quando a sessão tocar nesses arquivos/áreas:

- [checks/hero-and-lcp.md](checks/hero-and-lcp.md) — primeira dobra, imagem LCP, headline.
- [checks/images.md](checks/images.md) — formato, dimensões, lazy, preload, responsivo.
- [checks/fonts.md](checks/fonts.md) — Google Fonts, locais, `font-display`, preload, fallback.
- [checks/css.md](checks/css.md) — render-blocking, CSS crítico, unused CSS, Tailwind, Elementor CSS.
- [checks/javascript.md](checks/javascript.md) — defer/async, bundle, hydration, libs desnecessárias.
- [checks/third-party-tracking.md](checks/third-party-tracking.md) — Pixel, GTM, GA4, Hotmart, AC, Make, chats.
- [checks/wordpress-elementor.md](checks/wordpress-elementor.md) — plugins, assets globais, cache, LiteSpeed, Cloudflare.
- [checks/layout-stability.md](checks/layout-stability.md) — CLS, reservar espaço, fonts swap, embeds.
- [checks/network-server.md](checks/network-server.md) — cache, CDN, Brotli, TTFB, preconnect.

## Regras invioláveis

1. **Nunca recomendar lazy-load na imagem do LCP** — sempre `fetchpriority="high"` + sem `loading="lazy"`.
2. **Nunca remover ou mover scripts de tracking sem confirmar com o usuário** — pode quebrar atribuição.
3. **Sempre separar lab e field data** — não tratar Lighthouse score como verdade absoluta.
4. **Sempre informar mobile e desktop separadamente** — são rankings diferentes.
5. **Nunca recomendar `display: none` para esconder problema de CLS** — corrigir reservando espaço.
6. **Nunca dropar UTMs/query string** em otimizações de redirect ou canonical.
7. **Toda recomendação tem que apontar arquivo/linha quando aplicável** — sem "otimize o CSS" genérico.
8. **Toda análise tem mobile-first como default** — desktop é complemento.
9. **Relatório sempre gerado** — mesmo se tudo estiver verde.
10. **Score isolado não é critério de aprovação** — Core Web Vitals no field é.

## Quando NÃO usar

- Auditoria de SEO conteúdo/keywords → use `seo-audit`.
- Auditoria de acessibilidade profunda (WCAG AA/AAA) → skill dedicada de a11y.
- Auditoria de segurança de headers → `security-auditor` / `seguranca`.
- Bug de renderização visual (não relacionado a performance) → debug normal.
- Refactor de backend / API → fora de escopo.

## Skills relacionadas

- **congruence** — verifica se claims na landing batem com o que o código faz. Use depois de uma rodada de performance-audit que tenha alterado copy/CTA.
- **seo-audit** — auditoria de SEO técnico e on-page. Complementar.
- **ux-ui-perfectionist** — revisão de UI/UX. Use em conjunto se a otimização tocar layout.
- **ai-seo** — otimização para LLM search/AI Overviews. Complementar.

## Por que esta skill existe

Otimizar performance virou folclore: time troca framework, compra plugin, ativa "modo turbo" do CDN, e o score sobe 5 pontos enquanto a conversão cai 20%. Esta skill existe para **separar o que o Lighthouse mede do que o usuário sente**, e garantir que cada correção:

- Tem evidência concreta no relatório/código.
- Tem prioridade baseada em impacto × custo.
- Tem teste de regressão pra layout, tracking e funcionalidade.
- Não é cargo cult ("ouvi dizer que defer melhora").

O objetivo final não é o score. É **a página que carrega rápido pro usuário real, mantém a conversão e ainda passa nos Core Web Vitals**.
