# Use performance-audit with any LLM (ChatGPT, Claude.ai, Gemini, etc.)

> Paste-and-use prompt — no install, works in any chat interface.

## What you get

- Methodology activated for your current chat session
- 7-step workflow + 6-pillar guardrails enforced by the LLM
- Works on ChatGPT (web), Claude.ai, Gemini, Perplexity, Mistral Chat, DeepSeek, any LLM with chat UI

## Quick install

1. Open a fresh chat with your preferred LLM
2. Copy the entire content of [`performance-audit-prompt.md`](https://raw.githubusercontent.com/brunnocarpena/performance-audit-skill/main/adapters/generic-prompt/performance-audit-prompt.md)
3. Paste it as your first message
4. Continue normal conversation — the LLM will audit your page-performance changes

## How to invoke

After pasting:

```
I'm about to deploy this landing page: [paste diff]. PSI mobile: [paste]. Audit.
```

Or for an existing live page:

```
I want to ship this hero change: [paste]. Current PSI report: [paste]. CrUX: [paste]. Audit.
```

## Limitations vs Claude Code

- **No file system access** — the LLM cannot grep/read your repo. You must paste source files when asked.
- **No PSI runner** — paste lab/field data into chat.
- **No automation** — purely manual workflow.
- **Session-scoped** — methodology resets when you start a new chat (paste prompt again).
- **Best with reasoning models** — GPT-5, Claude Opus, Gemini 2.5 Pro, DeepSeek R1 produce better audits than smaller models.

## Updating

Re-copy the latest version of [`performance-audit-prompt.md`](https://raw.githubusercontent.com/brunnocarpena/performance-audit-skill/main/adapters/generic-prompt/performance-audit-prompt.md) any time.

## Reference

- Full skill: https://github.com/brunnocarpena/performance-audit-skill
