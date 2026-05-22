# Install performance-audit on Kimi CLI / Kimi K2

> **Status:** As of late 2025, Kimi CLI (MoonshotAI) does **not** have a documented project-memory file (no `KIMI.md` equivalent to `CLAUDE.md` / `GEMINI.md`). This adapter is **paste-per-session**.

## What you get

- Methodology activated for your current Kimi session
- 7-step workflow + 6-pillar guardrails enforced by Kimi K2
- Works with Kimi CLI or Kimi web chat

## Quick install

There's nothing to install. Just paste:

1. Open Kimi CLI: `kimi`
2. Copy the entire content of [`KIMI-PROMPT.md`](https://raw.githubusercontent.com/brunnocarpena/performance-audit-skill/main/adapters/kimi/KIMI-PROMPT.md)
3. Paste as your first message
4. Continue normal conversation

## How to invoke

After pasting:

```
I'm about to deploy this landing page: [paste diff or PSI report]. Audit.
```

## Limitations vs Claude Code

- **No persistent memory** — methodology must be re-pasted each new Kimi session.
- **No hooks, no Task tool** — fully manual workflow.
- **No PSI runner** — paste lab/field data into chat.
- **MCP available** — if you set up Kimi with MCP servers, you can give it file system tools, but the audit rules still need to be in the prompt.
- **If a `KIMI.md` convention emerges**, this adapter will be updated to support it.

## Reference

- Kimi CLI: https://github.com/MoonshotAI/kimi-cli
- Moonshot AI platform: https://platform.moonshot.ai/

> If you discover an updated Kimi project-memory convention, please open a PR.
