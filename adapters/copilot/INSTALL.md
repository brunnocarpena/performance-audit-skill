# Install performance-audit on GitHub Copilot

> Auto-injected on every Copilot Chat session in this repo. Ambient (no manual invocation needed).

## What you get

- Performance-audit methodology auto-included in Copilot's context for this repo
- 7-step workflow + 6-pillar guardrails before any deploy
- No manual invocation needed — Copilot reads `.github/copilot-instructions.md` automatically

## Quick install

```bash
cd your-project/
mkdir -p .github
curl -L https://raw.githubusercontent.com/xBelowZero/claude-skill-performance-audit/main/adapters/copilot/copilot-instructions.md \
  -o .github/copilot-instructions.md
```

Commit and push — Copilot picks it up immediately.

## How to invoke

Ambient. Just chat with Copilot as usual. The instructions are silently appended to every prompt.

To force the audit, ask explicitly: `audit page performance of these changes before I deploy`.

## Limitations vs Claude Code

- **No hooks** — Copilot has no Stop/PostToolUse equivalent. The audit relies on Copilot reading the instructions.
- **No Task tool** — to dispatch an independent auditor, open a new Copilot Chat session and paste the [auditor-prompt](https://github.com/xBelowZero/claude-skill-performance-audit/blob/main/auditor-prompt.md).
- **No PSI / Lighthouse runner** — Copilot can't fetch metrics on its own. Paste your PSI report or CrUX export into chat.
- **Repo-scoped** — instructions apply only to this repo.

## Updating

```bash
curl -L https://raw.githubusercontent.com/xBelowZero/claude-skill-performance-audit/main/adapters/copilot/copilot-instructions.md \
  -o .github/copilot-instructions.md
```

## Reference

- GitHub Copilot custom instructions: https://docs.github.com/en/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot
