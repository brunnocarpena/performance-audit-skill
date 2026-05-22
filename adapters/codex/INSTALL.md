# Install performance-audit on OpenAI Codex CLI

> Auto-parsed via `AGENTS.md` at project root.

## What you get

- Performance-audit methodology in every Codex session for this project
- 7-step workflow + 6-pillar guardrails before any deploy
- Nested `AGENTS.md` in subdirectories override root for that subtree

## Quick install

```bash
cd your-project/
curl -L https://raw.githubusercontent.com/brunnocarpena/performance-audit-skill/main/adapters/codex/AGENTS.md -o AGENTS.md
```

If you already have an `AGENTS.md`, append the performance-audit section manually rather than overwriting.

## How to invoke

Ambient. Codex auto-loads `AGENTS.md` from CWD and walks up.

## Limitations vs Claude Code

- **No hooks** — Codex CLI cannot auto-trigger the audit at end of turn.
- **No Task tool** — to dispatch a fresh auditor, open a new Codex session and paste [auditor-prompt](https://github.com/brunnocarpena/performance-audit-skill/blob/main/auditor-prompt.md).
- **No PSI runner** — paste lab/field data into chat.
- **AGENTS.md is shared** — if other tools (Cursor, Aider) also read it, the methodology applies cross-tool.

## Updating

```bash
curl -L https://raw.githubusercontent.com/brunnocarpena/performance-audit-skill/main/adapters/codex/AGENTS.md -o AGENTS.md
```

## Reference

- AGENTS.md spec: https://agents.md/
- OpenAI Codex CLI: https://github.com/openai/codex
