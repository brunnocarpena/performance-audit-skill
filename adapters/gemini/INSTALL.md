# Install performance-audit on Gemini CLI

> Auto-loaded into Gemini's hierarchical memory via `GEMINI.md`.

## What you get

- Methodology auto-included on every Gemini CLI session in this project
- 7-step workflow + 6-pillar guardrails
- `/memory show` and `/memory refresh` let you inspect/reload

## Quick install

Project-local (recommended):
```bash
cd your-project/
curl -L https://raw.githubusercontent.com/xBelowZero/claude-skill-performance-audit/main/adapters/gemini/GEMINI.md -o GEMINI.md
```

Global (all projects):
```bash
mkdir -p ~/.gemini
curl -L https://raw.githubusercontent.com/xBelowZero/claude-skill-performance-audit/main/adapters/gemini/GEMINI.md -o ~/.gemini/GEMINI.md
```

## How to invoke

Ambient. Gemini auto-discovers `GEMINI.md` walking up from CWD.

Verify it loaded: `/memory show` in Gemini CLI.

## Limitations vs Claude Code

- **No hooks** — Gemini CLI has no Stop hook to auto-trigger audit.
- **No Task tool** — to dispatch an independent auditor, open a new Gemini session and paste [auditor-prompt](https://github.com/xBelowZero/claude-skill-performance-audit/blob/main/auditor-prompt.md).
- **No PSI runner** — paste lab/field data into chat.
- **Customize filename** via `settings.json` if your project conflicts: `"context": { "fileName": "PERFORMANCE.md" }`.

## Updating

```bash
curl -L https://raw.githubusercontent.com/xBelowZero/claude-skill-performance-audit/main/adapters/gemini/GEMINI.md -o GEMINI.md
```

## Reference

- Gemini CLI docs: https://github.com/google-gemini/gemini-cli
- GEMINI.md format: https://www.geminicli.com/docs/cli/gemini-md
