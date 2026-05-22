# performance-audit

> Page performance auditing through the lens of **Google PageSpeed Insights / Lighthouse**, without sacrificing layout, tracking, UTMs, SEO or conversion.

🇺🇸 · [🇧🇷](README.pt-BR.md) · [🇪🇸](README.es.md) · [🇫🇷](README.fr.md) · [🇩🇪](README.de.md) · [🇨🇳](README.zh.md)

Skill built for Claude Code. Structurally inspired by the [congruence](https://github.com/xBelowZero/congruence-skill) skill — same pattern of Iron Law, mandatory evidence, explicit severity and pre-deploy decision.

---

## What it does

Performance audit that separates **what Lighthouse measures** from **what the user actually feels**, and prioritizes fixes with:

- Measurable impact on the right metric (LCP, CLS, INP, TBT, FCP).
- Realistic implementation cost.
- Explicit risk (breaks layout? breaks tracking? breaks conversion?).
- Post-fix test plan.

**It is not** blind optimization. Every recommendation preserves:

1. Visual layout.
2. Tracking (Pixel, GTM, GA4, Hotmart, AC, Make, conversions).
3. UTMs and URL parameters.
4. Form functionality.
5. SEO (meta, structured data, canonical, hreflang).
6. Basic accessibility.

## Operating modes

- **`audit`** — audit an existing page (URL, code, Lighthouse report).
- **`build`** — review code before deploy, ensuring performance from the start.

## When to use

Use **whenever**:
- You need to audit a page (landing, WP/Elementor, HTML/Tailwind, Next.js, capture).
- You're building a new page and want to guarantee a good score from the start.
- Before any public-page deploy.
- Investigating a Core Web Vitals regression.
- Investigating a conversion drop suspected to be speed-related.

Invoke with `/performance-audit` or "audit this page's performance".

## Installation

```bash
# 1. Clone into ~/.claude/skills/
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/xBelowZero/claude-skill-performance-audit.git performance-audit

# 2. (Optional but recommended) configure PageSpeed Insights API key
# Without key: ~1 query/day keyed by IP. With key: 400 QPS / 25k queries/day.
#
#   - https://console.cloud.google.com/apis/credentials → create API key
#   - Enable "PageSpeed Insights API" and "Chrome UX Report API"
#   - Add to ~/.zshrc (or ~/.bashrc):
#
#       export PSI_API_KEY="AIza..."
#       export CRUX_API_KEY="$PSI_API_KEY"
#
#   - source ~/.zshrc
```

Validate installation:

```bash
ls ~/.claude/skills/performance-audit/SKILL.md  # must exist
echo $PSI_API_KEY                                # should print the key (optional)
```

In any Claude Code session:

```
/performance-audit https://example.com --mode=audit --target=mobile
```

## Supported AI platforms

| Platform | Adapter | Status |
|----------|---------|--------|
| **Claude Code** (CLI / IDE / web) | root `SKILL.md` | ✅ Primary |
| **Cursor** | [`adapters/cursor/`](adapters/cursor/) | ✅ Supported |
| **GitHub Copilot** | [`adapters/copilot/`](adapters/copilot/) | ✅ Supported |
| **Gemini CLI / Antigravity** | [`adapters/gemini/`](adapters/gemini/) | ✅ Supported |
| **OpenAI Codex CLI** | [`adapters/codex/`](adapters/codex/) | ✅ Supported |
| **Aider** | [`adapters/aider/`](adapters/aider/) | ✅ Supported |
| **Qwen Code CLI** (Alibaba) | [`adapters/qwen/`](adapters/qwen/) | ✅ Supported |
| **Kimi K2** (Moonshot AI) | [`adapters/kimi/`](adapters/kimi/) | ✅ Supported |
| **Windsurf** (Codeium) | [`adapters/windsurf/`](adapters/windsurf/) | ✅ Supported |
| **Cline** (VS Code) | [`adapters/cline/`](adapters/cline/) | ✅ Supported |
| **Continue.dev** | [`adapters/continue/`](adapters/continue/) | ✅ Supported |
| **Zed Editor** | [`adapters/zed/`](adapters/zed/) | ✅ Supported |
| **JetBrains Junie** | [`adapters/jetbrains-junie/`](adapters/jetbrains-junie/) | ✅ Supported |
| **Sourcegraph Cody** | [`adapters/cody/`](adapters/cody/) | ✅ Supported |
| **ChatGPT Custom GPT** | [`adapters/chatgpt-custom-gpt/`](adapters/chatgpt-custom-gpt/) | ✅ Supported |
| Any other LLM | [`adapters/generic-prompt/`](adapters/generic-prompt/) | ✅ Copy-paste |

Each adapter folder has its own `INSTALL.md` with platform-specific install steps.

## When NOT to use

- SEO content audit → `seo-audit`.
- WCAG accessibility audit → dedicated skill.
- Security audit → `security-auditor`.
- Visual bug unrelated to performance.

## Structure

```
performance-audit/
├── SKILL.md                          # Main instructions, Iron Law, workflow (EN canonical)
├── SKILL.pt-BR.md                    # Portuguese (Brazil)
├── SKILL.es.md                       # Spanish
├── SKILL.fr.md                       # French
├── SKILL.de.md                       # German
├── SKILL.zh.md                       # Simplified Chinese
├── README.md / README.*.md           # This file in 6 languages
├── adapters/                         # 15 platform adapters
├── references/                       # Deep guides
│   ├── metrics-glossary.md           #   LCP/CLS/INP/TBT/FCP/SI/TTFB
│   ├── data-collection.md            #   How to collect lab + field data
│   ├── severity-rubric.md            #   Critical/high/medium/low rubric
│   ├── safe-fixes.md                 #   Fixes by risk class
│   ├── report-format.md              #   Report template
│   └── psi-lighthouse-internals.md   #   PSI/Lighthouse/CrUX deep mechanics
└── checks/                           # Domain checks
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

## How it works

1. Identifies scope and mode (audit vs build).
2. Collects evidence: Lighthouse + PageSpeed + CrUX + code.
3. Classifies into 12 categories (hero, images, fonts, CSS, JS, third-party, WP, CLS, network).
4. Assigns affected metric + severity + priority + risk.
5. Recommends **safe** fixes, always pointing to file/line and what NOT to touch alongside.
6. Emits a standardized report.
7. Decides pre-deploy: approved / with caveats / blocked.

## Inviolable rules

1. **The LCP image never has `loading="lazy"`** — always `fetchpriority="high"`.
2. **Tracking is not touched without explicit user confirmation**.
3. **Lab and field data always separate** — Lighthouse score is not absolute truth.
4. **Mobile and desktop always separate** — they're different Google rankings.
5. **UTMs are never dropped** in redirects/canonical.
6. **Every recommendation cites file/line or audit ID**.
7. **Mobile-first by default**.
8. **Report always emitted**, even when everything is green.
9. **An isolated score does not approve deploy** — Core Web Vitals "good" in field is the criterion.
10. **Every risky fix has a rollback plan**.

## Metrics and thresholds (2026)

| Metric | Good | Needs improvement | Poor |
|--------|------|--------------------|------|
| LCP | ≤ 2.5s | 2.5–4.0s | > 4.0s |
| CLS | ≤ 0.1 | 0.1–0.25 | > 0.25 |
| INP | ≤ 200ms | 200–500ms | > 500ms |
| TBT | ≤ 200ms | 200–600ms | > 600ms |
| FCP | ≤ 1.8s | 1.8–3.0s | > 3.0s |
| TTFB | ≤ 800ms | 800–1800ms | > 1800ms |

Google criterion: **p75 ≥ "good" for 75% of users in CrUX**.

## Inspiration and references

Structure inspired by:
- [xBelowZero/congruence-skill](https://github.com/xBelowZero/congruence-skill) — mandatory-evidence + Iron Law pattern.
- [addyosmani/web-quality-skills](https://github.com/addyosmani/web-quality-skills) — official Chrome team Agent Skills for Lighthouse + CWV.
- [web.dev/articles/vitals](https://web.dev/articles/vitals) — canonical source of thresholds.
- [GoogleChrome/lighthouse-ci](https://github.com/GoogleChrome/lighthouse-ci) — budgets and regressions in PR.

## Related skills

- **congruence** — verifies that page claims match the code. Use after if the audit modified copy/CTA.
- **seo-audit** — technical and on-page SEO. Complementary.
- **ux-ui-perfectionist** — UI/UX review. Use together if the optimization touches layout.

## License

MIT — see [LICENSE](LICENSE).
