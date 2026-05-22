# Install performance-audit on Aider

> Auto-loaded via `.aider.conf.yml` as cacheable read-only context.

## What you get

- Methodology read-only context in every Aider session
- 7-step workflow + 6-pillar guardrails
- Cacheable (low token cost on repeat sessions)

## Quick install

```bash
cd your-project/
curl -L https://raw.githubusercontent.com/brunnocarpena/performance-audit-skill/main/adapters/aider/CONVENTIONS.md -o CONVENTIONS.md
```

Then add to `.aider.conf.yml` at the project root (create if missing):

```yaml
read:
  - CONVENTIONS.md
```

Or for a single session:

```bash
aider --read CONVENTIONS.md
```

## How to invoke

Ambient. Aider treats the file as standing instructions.

## Limitations vs Claude Code

- **No hooks** — Aider has no Stop hook equivalent.
- **No Task tool** — to dispatch an independent auditor, open a new Aider session and paste [auditor-prompt](https://github.com/brunnocarpena/performance-audit-skill/blob/main/auditor-prompt.md).
- **No PSI runner** — paste lab/field data into chat.
- **No subagents** — audit is enforced by the same model handling implementation.

## Updating

```bash
curl -L https://raw.githubusercontent.com/brunnocarpena/performance-audit-skill/main/adapters/aider/CONVENTIONS.md -o CONVENTIONS.md
```

## Reference

- Aider conventions docs: https://aider.chat/docs/usage/conventions.html
