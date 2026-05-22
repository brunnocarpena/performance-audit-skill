# Install performance-audit on Continue.dev

> Auto-discovered from `.continue/rules/` directory.

## What you get

- Methodology in Continue's rules system
- 7-step workflow + 6-pillar guardrails
- Frontmatter `alwaysApply: false` — rule applies when context-relevant

## Quick install

```bash
cd your-project/
mkdir -p .continue/rules
curl -L https://raw.githubusercontent.com/xBelowZero/claude-skill-performance-audit/main/adapters/continue/performance-audit.md -o .continue/rules/performance-audit.md
```

## How to invoke

By default `alwaysApply: false` — Continue applies the rule when contextually relevant. To force always-on, edit frontmatter to `alwaysApply: true`.

Or use `globs:` field to scope:

```yaml
globs: ["**/*.html", "**/*.tsx", "**/*.jsx", "**/landing/**", "**/pages/**"]
```

## Limitations vs Claude Code

- **No hooks** — Continue has no Stop hook equivalent.
- **No Task tool** — for independent audit, open new Continue chat and paste [auditor-prompt](https://github.com/xBelowZero/claude-skill-performance-audit/blob/main/auditor-prompt.md).
- **No PSI runner** — paste lab/field data into chat.

## Updating

```bash
curl -L https://raw.githubusercontent.com/xBelowZero/claude-skill-performance-audit/main/adapters/continue/performance-audit.md -o .continue/rules/performance-audit.md
```

## Reference

- Continue.dev Rules: https://docs.continue.dev/customize/deep-dives/rules
