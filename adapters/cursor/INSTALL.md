# Install performance-audit on Cursor

> Audits page performance against Lighthouse / PageSpeed / Core Web Vitals while preserving layout, tracking, UTMs, forms, SEO, and a11y. Manual invocation via `@performance-audit`.

## What you get

- An Agent-Requested Cursor Rule that activates when you mention `@performance-audit`
- 7-step workflow (scope → evidence → classify → severity → recommend → report → pre-deploy decision)
- 6-pillar guardrails so optimizations never break tracking or conversion
- No automatic enforcement — Cursor has no Stop hook equivalent

## Quick install

```bash
cd your-project/
mkdir -p .cursor/rules
curl -L https://raw.githubusercontent.com/xBelowZero/claude-skill-performance-audit/main/adapters/cursor/performance-audit.mdc \
  -o .cursor/rules/performance-audit.mdc
```

Or clone the whole skill:

```bash
git clone https://github.com/xBelowZero/claude-skill-performance-audit.git /tmp/performance-audit
mkdir -p .cursor/rules
cp /tmp/performance-audit/adapters/cursor/performance-audit.mdc .cursor/rules/
```

## How to invoke

In Cursor chat:

```
@performance-audit audit this landing page before I deploy
```

Or include in any prompt where you're touching hero / images / fonts / scripts:

```
Optimize LCP on the homepage. Before shipping, follow @performance-audit.
```

## Limitations vs Claude Code

- **No Stop hook** — Cursor cannot auto-trigger the audit at end of turn. You must mention `@performance-audit`.
- **No Task tool** — to run a fresh second-opinion audit, open a new Cursor chat and paste the [auditor-prompt](https://github.com/xBelowZero/claude-skill-performance-audit/blob/main/auditor-prompt.md).
- **`globs` field** restricts where the rule applies — adjust the frontmatter glob list to match your front-end / landing-page paths.
- **No PSI / Lighthouse runner** — Cursor can't fetch field data on its own. Paste your PSI report or CrUX export into chat.

## Updating

```bash
curl -L https://raw.githubusercontent.com/xBelowZero/claude-skill-performance-audit/main/adapters/cursor/performance-audit.mdc \
  -o .cursor/rules/performance-audit.mdc
```

## Reference

- Cursor Rules docs: https://docs.cursor.com/context/rules
