# Performance Audit — Score 90+ no Lighthouse Sem Quebrar Tracking ou Conversão

> Auditoria de performance de páginas sob a ótica do **Google PageSpeed Insights / Lighthouse**, sem sacrificar layout, rastreamento, UTMs, SEO ou conversão.


---

## O que faz

Auditoria de performance que separa **o que o Lighthouse mede** de **o que o usuário sente**, e prioriza correções com:

- Impacto mensurável na métrica certa (LCP, CLS, INP, TBT, FCP).
- Custo de implementação realista.
- Risco explícito (quebra layout? quebra tracking? quebra conversão?).
- Plano de teste pós-correção.

**Não é** uma otimização cega. Toda recomendação preserva:

1. Layout visual.
2. Tracking (Pixel, GTM, GA4, Hotmart, AC, Make, conversões).
3. UTMs e parâmetros de URL.
4. Funcionalidade de formulários.
5. SEO (meta, structured data, canonical, hreflang).
6. Acessibilidade básica.

## Modos de operação

- **`audit`** — auditoria de página existente (URL, código, relatório Lighthouse).
- **`build`** — revisão de código antes de deploy, garantindo performance desde o início.

## Quando usar

Use **sempre que**:
- Precisar auditar uma página (landing, WP/Elementor, HTML/Tailwind, Next.js, captura).
- Estiver desenvolvendo página nova e quiser garantir boa pontuação desde o início.
- Antes de deploy de qualquer página pública.
- Investigando queda de Core Web Vitals.
- Investigando queda de conversão suspeita relacionada a velocidade.

Invocar com `/performance-audit` ou "audita performance dessa página".

## Instalação

```bash
# 1. Clonar dentro de ~/.claude/skills/
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/brunnocarpena/performance-audit-skill.git performance-audit

# 2. (Opcional mas recomendado) configurar API key do PageSpeed Insights
# Sem key: ~1 query/dia keyed por IP. Com key: 400 QPS / 25k queries/dia.
#
#   - https://console.cloud.google.com/apis/credentials → criar API key
#   - Habilitar "PageSpeed Insights API" e "Chrome UX Report API"
#   - Adicionar no ~/.zshrc (ou ~/.bashrc):
#
#       export PSI_API_KEY="AIza..."
#       export CRUX_API_KEY="$PSI_API_KEY"
#
#   - source ~/.zshrc
```

Validar instalação:

```bash
ls ~/.claude/skills/performance-audit/SKILL.md  # deve existir
echo $PSI_API_KEY                                # deve mostrar a key (opcional)
```

Em qualquer sessão Claude Code:

```
/performance-audit https://exemplo.com --mode=audit --target=mobile
```

## Quando NÃO usar

- Auditoria de SEO conteúdo → `seo-audit`.
- Auditoria de acessibilidade WCAG → skill dedicada.
- Auditoria de segurança → `security-auditor`.
- Bug visual sem relação com performance.

## Estrutura

```
performance-audit/
├── SKILL.md                          # Instruções principais, Iron Law, workflow
├── README.md                         # Este arquivo
├── references/                       # Guias profundos
│   ├── metrics-glossary.md           #   LCP/CLS/INP/TBT/FCP/SI/TTFB
│   ├── data-collection.md            #   Como coletar lab + field data
│   ├── severity-rubric.md            #   Rubrica crítica/alta/média/baixa
│   ├── safe-fixes.md                 #   Correções por risco
│   └── report-format.md              #   Template do relatório
└── checks/                           # Auditoria por domínio
    ├── hero-and-lcp.md
    ├── images.md
    ├── fonts.md
    ├── css.md
    ├── javascript.md
    ├── third-party-tracking.md
    ├── wordpress-elementor.md
    ├── layout-stability.md
    └── network-server.md
```

## Como funciona

1. Identifica escopo e modo (audit vs build).
2. Coleta evidência: Lighthouse + PageSpeed + CrUX + código.
3. Classifica em 12 categorias (hero, imagens, fonts, CSS, JS, third-party, WP, CLS, rede).
4. Atribui métrica afetada + severidade + prioridade + risco.
5. Recomenda correções **seguras**, sempre indicando arquivo/linha e o que NÃO mexer junto.
6. Emite relatório padronizado.
7. Decide pré-deploy: aprovado / com ressalvas / bloqueado.

## Regras invioláveis

1. **Imagem do LCP nunca tem `loading="lazy"`** — sempre `fetchpriority="high"`.
2. **Tracking não se mexe sem confirmação explícita do usuário**.
3. **Lab e field data sempre separados** — score Lighthouse não é verdade absoluta.
4. **Mobile e desktop sempre separados** — são rankings diferentes do Google.
5. **UTMs nunca são dropados** em redirects/canonical.
6. **Toda recomendação cita arquivo/linha ou audit ID**.
7. **Mobile-first por default**.
8. **Relatório sempre gerado**, mesmo se tudo estiver verde.
9. **Score isolado não aprova deploy** — Core Web Vitals "good" no field é o critério.
10. **Toda correção arriscada tem plano de rollback**.

## Métricas e thresholds (2026)

| Métrica | Bom | Precisa melhorar | Ruim |
|---------|-----|------------------|------|
| LCP | ≤ 2.5s | 2.5–4.0s | > 4.0s |
| CLS | ≤ 0.1 | 0.1–0.25 | > 0.25 |
| INP | ≤ 200ms | 200–500ms | > 500ms |
| TBT | ≤ 200ms | 200–600ms | > 600ms |
| FCP | ≤ 1.8s | 1.8–3.0s | > 3.0s |
| TTFB | ≤ 800ms | 800–1800ms | > 1800ms |

Critério Google: **p75 ≥ "good" em 75% dos usuários no CrUX**.

## Inspiração e referências

Estrutura inspirada em:
- [brunnocarpena/congruence-skill](https://github.com/brunnocarpena/congruence-skill) — padrão de evidência obrigatória + Iron Law.
- [addyosmani/web-quality-skills](https://github.com/addyosmani/web-quality-skills) — Agent Skills oficiais do time Chrome para Lighthouse + CWV.
- [web.dev/articles/vitals](https://web.dev/articles/vitals) — fonte canônica dos thresholds.
- [GoogleChrome/lighthouse-ci](https://github.com/GoogleChrome/lighthouse-ci) — budgets e regressões em PR.

## Skills relacionadas

- **congruence** — verifica se claims da página batem com o código. Use depois se a auditoria mexer em copy/CTA.
- **seo-audit** — SEO técnico e on-page. Complementar.
- **ux-ui-perfectionist** — revisão de UI/UX. Use em conjunto se a otimização tocar layout.
