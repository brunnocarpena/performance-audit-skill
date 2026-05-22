# Formato do Relatório de Auditoria de Performance

Template **obrigatório** para o relatório gerado pela skill. Sempre seguir esta estrutura.

---

```markdown
# Auditoria de Performance

**Página:** [URL ou nome do arquivo]
**Data:** [YYYY-MM-DD]
**Modo:** [audit / build]
**Target:** [mobile / desktop / both]
**Fontes de evidência:** [Lighthouse run / PageSpeed / CrUX / código]

---

## Diagnóstico rápido

[2–4 frases: estado geral, principais gargalos, decisão pré-deploy esperada.]

---

## Métricas

### Field data (CrUX, p75 últimos 28 dias)

| Métrica | Mobile | Desktop | Status |
|---------|--------|---------|--------|
| LCP | X.Xs | X.Xs | good/needs/poor |
| CLS | 0.XX | 0.XX | good/needs/poor |
| INP | XXXms | XXXms | good/needs/poor |
| TTFB | XXXms | XXXms | good/needs/poor |

> Se field data indisponível, marcar "sem dados de campo" e justificar pela lab.

### Lab data (Lighthouse)

| Métrica | Mobile | Desktop |
|---------|--------|---------|
| Performance Score | XX | XX |
| FCP | X.Xs | X.Xs |
| LCP | X.Xs | X.Xs |
| TBT | XXXms | XXXms |
| CLS | 0.XX | 0.XX |
| Speed Index | X.Xs | X.Xs |

---

## Principais gargalos (top 5)

1. **[Nome do problema]** — afeta [LCP/CLS/INP/...]. Evidência: `arquivo:linha` ou audit Lighthouse `audit-id`.
2. ...
3. ...
4. ...
5. ...

---

## Tabela de impacto

| # | Problema | Métrica | Gravidade | Prioridade | Custo de correção | Risco |
|---|----------|---------|-----------|------------|-------------------|-------|
| 1 | Imagem hero 1.2MB sem WebP | LCP | crítica | P0 | baixo | baixo |
| 2 | GTM síncrono no `<head>` | TBT/INP | alta | P0 | médio | **alto (tracking)** |
| 3 | Fonte custom sem preload | LCP/CLS | média | P1 | baixo | baixo |
| ... | | | | | | |

**Legenda:**
- Gravidade: crítica / alta / média / baixa
- Prioridade: P0 (bloqueia deploy) / P1 (próximo sprint) / P2 (backlog)
- Custo: baixo / médio / alto (esforço de implementação)
- Risco: baixo / médio / alto (chance de quebrar layout, tracking, conversão)

---

## Correções recomendadas

### Prioridade Alta (P0)

#### 1. [Nome do problema]

**Métrica afetada:** [LCP / CLS / INP / ...]
**Audit Lighthouse:** `audit-id` (se aplicável)
**Arquivo:** `caminho/do/arquivo:linha`

**Problema:**
[Descrição objetiva — o que está acontecendo e por que afeta a métrica.]

**Correção:**
```html
<!-- ANTES -->
<img src="hero.jpg" loading="lazy">

<!-- DEPOIS -->
<img src="hero.webp" width="1200" height="600"
     fetchpriority="high" decoding="async" alt="...">
```

**Impacto esperado:** [LCP -1.2s, ou TBT -300ms, ou similar]

**O que NÃO mexer junto:**
- Não trocar o caminho do `src` se o tracking de assets do Pixel/GTM referencia ele.
- Não remover `alt` (SEO/a11y).
- Preservar UTMs em qualquer redirect.

**Como testar depois:**
1. Rodar Lighthouse mobile 3x (mediana).
2. Verificar CrUX em 28 dias.
3. Regressão visual no hero (screenshot diff).
4. Confirmar tracking: Pixel/GA4 ainda recebem `PageView`.

---

### Prioridade Média (P1)

[Mesma estrutura, abreviada.]

---

### Prioridade Baixa (P2)

[Mesma estrutura, mais sucinta — uma linha por item se possível.]

---

## Riscos ao corrigir

Lista de pontos que exigem cuidado especial:

- [ ] Mover GTM para o `</body>` quebra trigger "Page View" se estiver em "Page View - DOM Ready" mal configurado.
- [ ] Preload de fonte errada (formato woff em vez de woff2) gera warning + duplica download.
- [ ] Lazy-load em iframe do Vimeo pode quebrar autoplay em embed.
- [ ] Remover plugin de cache sem reconfigurar headers de cache no servidor → TTFB sobe.
- [ ] ...

---

## Checklist pré-deploy

- [ ] Lighthouse mobile rodado 3× — mediana coletada.
- [ ] Lighthouse desktop rodado 3× — mediana coletada.
- [ ] PageSpeed Insights validado em ambos.
- [ ] CrUX consultado (se houver tráfego suficiente).
- [ ] Regressão visual no hero/CTAs/forms confirmada.
- [ ] Tracking (Pixel, GA4, GTM, conversões) testado em modo preview.
- [ ] UTMs preservados em redirects/canonical.
- [ ] Formulários submetem corretamente.
- [ ] CTAs levam aos destinos corretos.
- [ ] Mobile testado em device real (não só DevTools).
- [ ] Cache do CDN purgado.

---

## Testes recomendados pós-deploy

1. **Lighthouse CI** em 24h — comparar com baseline.
2. **CrUX dashboard** em 7 dias e 28 dias.
3. **RUM** (web-vitals lib / useReportWebVitals no Next) — INP real.
4. **Funil de conversão** — comparar com baseline (não deve cair).
5. **Eventos de tracking** — Pixel/GA4/conversões batendo com baseline.

---

## Decisão pré-deploy

[**APROVADO** / **APROVADO COM RESSALVAS** / **BLOQUEADO** / **REVISÃO MANUAL NECESSÁRIA**]

**Justificativa:**
[Por que essa decisão. Quais problemas críticos restam (se houver).]

**Próximas ações (em ordem):**
1. ...
2. ...
3. ...
```

---

## Regras do relatório

1. **Sempre gerar**, mesmo se tudo estiver verde.
2. **Mobile e desktop separados** — nunca um único score consolidado.
3. **Field data > lab data** — quando ambos existem, decidir pelo field.
4. **Evidência concreta** — arquivo/linha ou audit ID. Sem "provavelmente é a imagem".
5. **Correção acionável** — sempre dizer o que mudar e onde, com exemplo de código quando aplicável.
6. **Riscos explícitos** — toda correção lista o que NÃO mexer junto.
7. **Decisão obrigatória** — nunca terminar sem a seção "Decisão pré-deploy".
8. **Limites quantitativos** — top 5 gargalos, não top 20. Priorização > exaustividade.
