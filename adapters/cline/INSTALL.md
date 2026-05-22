# Install performance-audit on Cline (VS Code extension)

> Auto-loaded via `.clinerules/` directory. Always active unless toggled off in Cline UI.

## What you get

- Methodology auto-included on every Cline session in this project
- 7-step workflow + 6-pillar guardrails
- Toggleable per-session via Cline's rules UI

## Quick install

```bash
cd your-project/
mkdir -p .clinerules
curl -L https://raw.githubusercontent.com/brunnocarpena/performance-audit-skill/main/adapters/cline/performance-audit.md -o .clinerules/performance-audit.md
```

## How to invoke

Ambient. Cline auto-loads all files in `.clinerules/`.

## Limitations vs Claude Code

- **No hooks** — Cline doesn't have Stop hook equivalent.
- **No Task tool** — to dispatch a fresh auditor, open a new Cline conversation and paste [auditor-prompt](https://github.com/brunnocarpena/performance-audit-skill/blob/main/auditor-prompt.md).
- **No PSI runner** — paste lab/field data into chat.
- **Multiple rules** can coexist in `.clinerules/` — performance-audit won't conflict with others.

## Updating

```bash
curl -L https://raw.githubusercontent.com/brunnocarpena/performance-audit-skill/main/adapters/cline/performance-audit.md -o .clinerules/performance-audit.md
```

## Reference

- Cline Rules: https://docs.cline.bot/features/cline-rules
