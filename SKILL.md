---
name: performance-audit
description: Use whenever you need to audit the performance of a page (landing page, WordPress/Elementor, HTML/Tailwind, Next.js, capture page) through the lens of Google PageSpeed Insights / Lighthouse, OR when you're building a new page and want to guarantee a good score from the start. TRIGGER when the agent mentions "PageSpeed", "Lighthouse", "Core Web Vitals", "LCP", "CLS", "INP", "TBT", "performance", "slow page", "low score", "page optimization"; when the session touches hero, image, font, critical CSS, GTM/Pixel/Analytics, cache plugin or Elementor files; before any deploy of a public page. SKIP for purely internal changes (API, jobs, schema) with no front-end render impact.
argument-hint: [url-or-scope?] [--mode=audit|build] [--target=mobile|desktop|both]
allowed-tools: Read, Grep, Glob, Bash(curl:*), Bash(git diff:*), Bash(git status:*), Bash(ls:*), Bash(find:*), Bash(wc:*), WebFetch, Task
---

# Performance Audit

Page performance auditing through the lens of **Google PageSpeed Insights / Lighthouse**, without sacrificing layout, tracking, UTMs, SEO or conversion.

**This skill is not blind optimization.** It is a critical audit that separates what Lighthouse measures from what the user actually feels, and prioritizes fixes that deliver real impact without breaking the page.

## Core principle

```
PERFORMANCE IS NOT A SCORE. IT IS DELIVERING THE FIRST FOLD FAST,
KEEPING VISUAL STABILITY, AND NOT BREAKING TRACKING OR CONVERSION.
```

A 100 score with a broken form, a Pixel that doesn't fire, or a lost UTM **is a regression**, not a win. Every recommendation in this skill must preserve:

1. **Visual layout** — no introduced CLS, no broken grid/components.
2. **Tracking** — Meta Pixel, GTM, GA4, Hotmart, ActiveCampaign, Make, conversions.
3. **UTMs and URL parameters** — never drop query string in redirects/canonical.
4. **Form functionality** — fields, validation, action, integrations.
5. **SEO** — meta tags, structured data, hreflang, canonical, robots.
6. **Basic accessibility** — alt, labels, contrast, focus.

If an "optimization" puts any of these at risk, **do not recommend it without an explicit warning**.

## Operating modes

The skill operates in two modes. Identify the mode upfront:

### `audit` mode — auditing an existing page

Accepted inputs:
- Public URL.
- PageSpeed Insights screenshot/JSON.
- Lighthouse report (HTML/JSON).
- Source code (HTML, Elementor template, Next.js component).
- Specific snippet (hero, head, footer).

Output: prioritized diagnosis + fix plan (see [references/report-format.md](references/report-format.md)).

### `build` mode — developing a new page

Apply best practices **before** code exists. Review HTML/CSS/JS at commit time. Validate hero, images, fonts, scripts and tracking against the checks in [checks/](checks/).

Output: pre-deploy checklist + risk notes on current code.

## Workflow

### 1. Identify scope and mode

- If there's a URL or Lighthouse report → `audit`.
- If editing page/component files without a report → `build`.
- If ambiguous, ask **once**: "Do you want to audit an existing page or review code before deploy?"

### 2. Collect evidence

**Mandatory pre-check — PageSpeed Insights API key**

Before any collection, verify `$PSI_API_KEY` is configured:

```bash
if [ -z "$PSI_API_KEY" ]; then
  echo "PSI_API_KEY missing"
fi
```

**If the variable is NOT set, pause and warn the user with this message** (adapt tone to conversation, keep the content):

> ⚠️ You haven't configured the PageSpeed Insights API key (`$PSI_API_KEY`) yet. Without it I'm limited to ~1 query/day keyed by IP — quota hits fast and I'm forced to do static-only audit (no score, no CrUX field data, no Lighthouse audits).
>
> **With the key configured, I can be much more accurate:** real Performance Score, measured LCP/CLS/INP/TBT/FCP, prioritized opportunities, main thread breakdown, heavy script identification, and field data (CrUX) from real users in the last 28 days.
>
> Setup (2 min):
> 1. https://console.cloud.google.com/apis/credentials → create API key
> 2. Enable **PageSpeed Insights API** and **Chrome UX Report API** under APIs & Services → Library
> 3. Add to `~/.zshrc`:
>    ```bash
>    export PSI_API_KEY="AIza..."
>    export CRUX_API_KEY="$PSI_API_KEY"
>    ```
> 4. `source ~/.zshrc` or open a new terminal
>
> Want to set it up now, or shall I proceed with limited static audit?

If the user opts to configure: wait and validate with a `curl` test. If they opt to proceed without: mark the final report as **"Partial audit — no PSI/CrUX data"** and base conclusions on static analysis only (HTML/headers/assets).

**Effective collection:**

| Source | What to extract |
|--------|------------------|
| Public URL | Run `curl -sI` for headers, download HTML, inspect `<head>`, script order, hero images |
| Lighthouse report | Metrics (LCP/CLS/INP/TBT/FCP/SI), top opportunities, diagnostics, JS treemap |
| Code | Hero, `<link>`/`<script>` order, image attributes, fonts, critical CSS, plugins (WP), bundle (Next) |
| PageSpeed Insights | Lab data **and** field data (CrUX) separately, mobile **and** desktop separately |

For WebFetch / curl on a public URL, see [references/data-collection.md](references/data-collection.md). For PSI/Lighthouse/CrUX internals (lab×field, score weights, INP, soft-nav, throttling, PSI API v5), see [references/psi-lighthouse-internals.md](references/psi-lighthouse-internals.md).

### 3. Classify by category

Every analysis is organized into **12 categories** (see [checks/](checks/)):

1. Executive summary
2. PageSpeed metrics (LCP, CLS, INP/TBT, FCP, SI)
3. First fold (hero, headline, CTA, critical CSS)
4. Images
5. Fonts
6. CSS
7. JavaScript
8. Third-parties and tracking
9. WordPress / Elementor
10. Visual stability (CLS)
11. Network and server
12. Action plan

### 4. Assign impact and severity

Every problem gets:
- **Affected metric** (LCP, CLS, INP, TBT, FCP, SI, TTFB).
- **Severity** (critical / high / medium / low) — see [references/severity-rubric.md](references/severity-rubric.md).
- **Priority** (P0 / P1 / P2) — based on impact × fix cost.
- **Fix risk** (breaks layout? breaks tracking? requires visual regression?).

### 5. Recommend a safe fix

For each problem:
- What exactly to change (file / selector / attribute).
- Why this affects metric X.
- What NOT to touch alongside.
- How to test afterwards (Lighthouse run, WebPageTest, DevTools Performance, visual regression).

Safe vs risky fix catalog: [references/safe-fixes.md](references/safe-fixes.md).

### 6. Emit the report

Mandatory template: [references/report-format.md](references/report-format.md).

### 7. Pre-deploy decision

- Any unresolved **critical** problem → **blocks deploy**.
- Multiple **highs** on hero/LCP → **blocks** until the top 3 are fixed.
- Only **medium/low** → **approved with caveats**.
- All green + Core Web Vitals within threshold → **approved**.

## Metrics and thresholds (quick reference)

2026 Core Web Vitals values (aligned with Google CrUX). Always validate against [web.dev/vitals](https://web.dev/articles/vitals) if in doubt.

| Metric | Good | Needs improvement | Poor |
|--------|------|--------------------|------|
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | 2.5s – 4.0s | > 4.0s |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | 0.1 – 0.25 | > 0.25 |
| **INP** (Interaction to Next Paint) | ≤ 200ms | 200ms – 500ms | > 500ms |
| **TBT** (Total Blocking Time, lab) | ≤ 200ms | 200ms – 600ms | > 600ms |
| **FCP** (First Contentful Paint) | ≤ 1.8s | 1.8s – 3.0s | > 3.0s |
| **Speed Index** | ≤ 3.4s | 3.4s – 5.8s | > 5.8s |
| **TTFB** | ≤ 800ms | 800ms – 1800ms | > 1800ms |

Details in [references/metrics-glossary.md](references/metrics-glossary.md).

## Lab Data vs. Field Data — ALWAYS differentiate

- **Lab data (Lighthouse)** — simulation on fixed throttling (Moto G4 / slow 4G). Useful for diagnosis.
- **Field data (CrUX)** — real Chrome users in the last 28 days. Useful for real prioritization.
- **Common discrepancy:** Lighthouse shows LCP 4s but CrUX shows 2.1s — real user is fine, Lighthouse score isn't. **Decide based on field, validate with lab.**
- Mobile and desktop **have separate Google rankings**. Always evaluate both.

## Red Flags — STOP immediately

If you catch yourself thinking or recommending:

- "Defer all scripts" — without mapping tracking/inline dependencies.
- "Lazy-load every image" — including the LCP image.
- "Remove the Pixel/GTM because it's heavy" — without aligning with marketing.
- "Replace Elementor with native code" — without understanding the editing cycle.
- "Compress the hero down to 30KB" — losing headline visual quality.
- "Remove unused CSS automatically" — may break states (hover, modal, error).
- "Lighthouse score 100 is the goal" — the goal is Core Web Vitals "good" in field data.
- "Migrate to Next.js / framework X solves it" — without understanding the real problem.

→ STOP. Go back to step 4 (impact × risk) before recommending.

## Excuse | Reality

| Excuse | Reality |
|--------|---------|
| "Lighthouse only runs on slow mobile, it's not real" | It's the ranking Google uses. Field data corroborates. |
| "Score 80 is good enough" | Core Web Vitals is binary (pass/fail). 80 can fail LCP. |
| "I optimized everything and the score dropped" | You probably broke inline tracking or LCP. Re-audit. |
| "Client wants 100" | 100 is fragile. Goal is stable "all green CWV". Educate the client. |
| "I'll use plugin X that does everything" | Plugins do 60% and break 30%. Audit the result, not the plugin. |
| "Mobile doesn't matter, my audience is desktop" | Google's mobile-first index ranks by mobile. |
| "I removed custom fonts and it got fast" | Conversion dropped too. Optimize the font, don't remove. |

## Domain checks

Use the appropriate check file when the session touches these files/areas:

- [checks/hero-and-lcp.md](checks/hero-and-lcp.md) — first fold, LCP image, headline.
- [checks/images.md](checks/images.md) — format, dimensions, lazy, preload, responsive.
- [checks/fonts.md](checks/fonts.md) — Google Fonts, local fonts, `font-display`, preload, fallback.
- [checks/css.md](checks/css.md) — render-blocking, critical CSS, unused CSS, Tailwind, Elementor CSS.
- [checks/javascript.md](checks/javascript.md) — defer/async, bundle, hydration, unnecessary libs.
- [checks/third-party-tracking.md](checks/third-party-tracking.md) — Pixel, GTM, GA4, Hotmart, AC, Make, chats.
- [checks/wordpress-elementor.md](checks/wordpress-elementor.md) — plugins, global assets, cache, LiteSpeed, Cloudflare.
- [checks/layout-stability.md](checks/layout-stability.md) — CLS, reserving space, fonts swap, embeds.
- [checks/network-server.md](checks/network-server.md) — cache, CDN, Brotli, TTFB, preconnect.

## Inviolable rules

1. **Never recommend lazy-load on the LCP image** — always `fetchpriority="high"` + no `loading="lazy"`.
2. **Never remove or move tracking scripts without confirming with the user** — may break attribution.
3. **Always separate lab and field data** — don't treat Lighthouse score as absolute truth.
4. **Always report mobile and desktop separately** — they're different rankings.
5. **Never recommend `display: none` to hide a CLS problem** — fix it by reserving space.
6. **Never drop UTMs/query string** in redirect or canonical optimizations.
7. **Every recommendation must point to a file/line when applicable** — no generic "optimize the CSS".
8. **Every analysis is mobile-first by default** — desktop is complement.
9. **Always emit the report** — even when everything is green.
10. **An isolated score is not approval criteria** — Core Web Vitals in field is.

## When NOT to use

- SEO content/keyword auditing → use `seo-audit`.
- Deep accessibility auditing (WCAG AA/AAA) → dedicated a11y skill.
- Security header auditing → `security-auditor` / `seguranca`.
- Visual rendering bug (not performance-related) → regular debug.
- Backend / API refactor → out of scope.

## Related skills

- **congruence** — checks if landing page claims match what the code does. Use after a `performance-audit` round that touched copy/CTA.
- **seo-audit** — technical and on-page SEO audit. Complementary.
- **ux-ui-perfectionist** — UI/UX review. Use together if the optimization touches layout.
- **ai-seo** — optimization for LLM search/AI Overviews. Complementary.

## Why this skill exists

Optimizing performance has become folklore: teams swap frameworks, buy plugins, turn on the CDN's "turbo mode", and the score goes up 5 points while conversion drops 20%. This skill exists to **separate what Lighthouse measures from what the user feels**, and ensure every fix:

- Has concrete evidence in the report/code.
- Has priority based on impact × cost.
- Has a regression test for layout, tracking and functionality.
- Is not cargo cult ("I heard defer makes it faster").

The final goal is not the score. It is **the page that loads fast for the real user, keeps converting, and still passes Core Web Vitals**.
