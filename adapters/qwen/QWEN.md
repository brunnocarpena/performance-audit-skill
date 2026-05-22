# Performance Audit

Audits page performance through the lens of Google PageSpeed Insights / Lighthouse / Core Web Vitals — and **separates what the lab measures from what the user feels**. Not a general code review.

**Core principle:** A high Lighthouse score is not the goal. Delivering the first fold fast, keeping the layout stable, and not breaking tracking or conversion is.

**Violating the letter of this rule is violating the spirit of it.** A "score win" that breaks a pillar is not a win.

## The Iron Law

```
PERFORMANCE IS NOT A SCORE. IT'S DELIVERING THE FIRST FOLD FAST,
KEEPING VISUAL STABILITY, AND NOT BREAKING TRACKING OR CONVERSION.
```

If a "fix" raises the score by 10 points but breaks GA4, the Meta Pixel, the form submit, or the hero CTA — that fix is rejected.

## The 6 pillars every recommendation must preserve

1. **Layout** — no visible reflow, no shifted hero, no broken responsive breakpoints
2. **Tracking** — GA4, GTM, Meta Pixel, server-side tags, conversion events all firing
3. **UTMs / attribution** — query strings preserved, referrer not stripped, redirects keep params
4. **Forms** — submit works, validation works, thank-you / redirect flow intact
5. **SEO** — meta tags, canonical, OG/Twitter cards, structured data, internal links
6. **Basic a11y** — focus visible, contrast preserved, alt text on real content images

## Workflow

1. **SCOPE** — what page, what device profile (mobile vs desktop), what stack (WordPress/Elementor, Next.js, plain HTML, etc.)
2. **EVIDENCE** — collect lab data (Lighthouse/PSI run) AND field data (CrUX) — separately, never mixed
3. **CLASSIFY** — bucket each issue: render-blocking, image weight, JS execution, layout shift, third-party, font loading, CSS unused, cache headers
4. **SEVERITY** — Critical / High / Medium / Low per Core Web Vital impact
5. **RECOMMEND** — concrete fix + which of the 6 pillars it touches + rollback plan
6. **REPORT** — mobile + desktop separated; lab + field separated; per-issue: metric / severity / priority / risk
7. **PRE-DEPLOY DECISION** — approved / approved with reservations / blocked

## Target metrics (field, mobile)

- **LCP** ≤ 2.5s
- **CLS** ≤ 0.1
- **INP** ≤ 200ms
- **TBT** (lab) ≤ 200ms as proxy for INP when field data is missing

Score alone never approves a deploy. Core Web Vitals "good" on **field** is the criterion.

## Severity rubric

- **Critical (P0)** — blocks Core Web Vitals "good", or breaks a pillar (tracking down, form broken, layout collapsed)
- **High (P1)** — single CWV stuck in "needs improvement", or significant regression on a pillar
- **Medium (P2)** — wasted bytes, second-order opportunities, no CWV impact
- **Low** — nice-to-have, cosmetic, opinionated micro-optimizations

## Red flags — STOP

- Lazy-loading the LCP image (above-the-fold hero) — never
- Removing tracking/pixel/GTM "to improve score" without explicit product confirmation
- Deferring JS that owns the CTA / form / interactive hero
- Stripping UTM params during a redirect chain
- Inlining critical CSS without testing the dark-mode / responsive variants
- "Score went from 60 to 95" with no field data confirmation
- Replacing real images with placeholders and calling it an LCP fix
- Editing `wp-content/plugins/elementor/**` core files instead of customizer/snippets

## Full methodology

For the full workflow (scoping rubric, issue catalog by stack, report template, rollback playbooks): https://github.com/xBelowZero/claude-skill-performance-audit/blob/main/SKILL.md


## Qwen Code specifics

Qwen Code (fork of Gemini CLI for Qwen3-Coder) auto-discovers `QWEN.md` hierarchically. Verify with `/memory show`. Reload via `/memory refresh`.
