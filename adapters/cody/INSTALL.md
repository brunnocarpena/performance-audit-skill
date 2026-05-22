# Install performance-audit on Sourcegraph Cody

> Cody has no `.cody/rules/` convention — performance-audit is installed as a **saved Prompt** in Cody's Prompt Library, invoked manually.

## What you get

- Manually-invoked `/performance-audit` command in Cody Chat
- 7-step workflow + 6-pillar guardrails on demand
- Workspace alternative via `.vscode/cody.json`

## Quick install (Prompt Library, recommended)

1. Open Sourcegraph instance → Cody panel → Prompt Library
2. Create new prompt:
   - Name: `performance-audit`
   - Description: `Audit page performance against Lighthouse/PSI/CWV before deploy`
   - Definition: paste content of [`cody-prompt.md`](https://raw.githubusercontent.com/xBelowZero/claude-skill-performance-audit/main/adapters/cody/cody-prompt.md)
3. Save

## Quick install (workspace command, alternative)

Add to `.vscode/cody.json`:

```json
{
  "commands": {
    "performance-audit": {
      "description": "Audit page performance against CWV",
      "prompt": "[paste cody-prompt.md content here]",
      "context": { "currentDir": true }
    }
  }
}
```

## How to invoke

In Cody Chat:

```
/performance-audit audit this landing page before I deploy
```

## Limitations vs Claude Code

- **No ambient activation** — must invoke explicitly each time.
- **No hooks, no Task tool**.
- **No PSI runner** — paste lab/field data into chat.
- **Prompt Library is per-instance** — share within your Sourcegraph organization.
- **No file system access from prompts** — Cody reads what's in chat context.

## Updating

Edit the saved prompt in Prompt Library with the latest content from [`cody-prompt.md`](https://raw.githubusercontent.com/xBelowZero/claude-skill-performance-audit/main/adapters/cody/cody-prompt.md).

## Reference

- Cody Prompts: https://sourcegraph.com/docs/cody/capabilities/prompts
- Prompt Library: https://sourcegraph.com/blog/announcing-prompt-library
