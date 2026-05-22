# Install performance-audit on Zed editor

> Auto-included in Zed's Agent Panel via `.rules` at project root.

## What you get

- Methodology in Zed's Agent context
- 7-step workflow + 6-pillar guardrails
- Works with any model Zed supports (Claude, GPT, Gemini, local)

## Quick install

```bash
cd your-project/
curl -L https://raw.githubusercontent.com/brunnocarpena/performance-audit-skill/main/adapters/zed/.rules -o .rules
```

> If you already have a `.rules` file, append the performance-audit section manually.

## How to invoke

Ambient. Zed's Agent Panel auto-includes `.rules`.

Zed also accepts these alternative filenames (first match wins):
- `.rules` (preferred)
- `.cursorrules`
- `AGENTS.md`
- `CLAUDE.md`
- `GEMINI.md`

If you already use one of those, performance-audit content can go there instead.

## Limitations vs Claude Code

- **No hooks** — Zed has no Stop hook equivalent.
- **No Task tool** — for independent audit, open new Zed Agent thread and paste [auditor-prompt](https://github.com/brunnocarpena/performance-audit-skill/blob/main/auditor-prompt.md).
- **No PSI runner** — paste lab/field data into chat.

## Updating

```bash
curl -L https://raw.githubusercontent.com/brunnocarpena/performance-audit-skill/main/adapters/zed/.rules -o .rules
```

## Reference

- Zed AI Rules: https://zed.dev/docs/ai/rules
