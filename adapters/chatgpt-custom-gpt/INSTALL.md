# Install performance-audit as a ChatGPT Custom GPT

> Create a Custom GPT named "Performance Auditor" — paste the prepared instructions into GPT Builder.

## What you get

- A dedicated ChatGPT GPT that audits page performance against Lighthouse/PSI/CWV
- Available in your ChatGPT account anywhere (web, mobile, desktop)
- Optionally shareable with your team or public

## Quick install

1. Open https://chatgpt.com/gpts/editor
2. Tab "Configure"
3. Name: `Performance Auditor`
4. Description: `Audits page performance against Lighthouse/PSI/CWV while preserving layout, tracking, UTMs, forms, SEO, a11y`
5. Instructions field: paste the entire content of [`instructions.md`](https://raw.githubusercontent.com/brunnocarpena/performance-audit-skill/main/adapters/chatgpt-custom-gpt/instructions.md)
6. Conversation starters (optional):
   - "Audit this landing page before I deploy"
   - "Check this PSI report and recommend fixes"
   - "I'm about to lazy-load my hero image — review the change"
7. Capabilities: enable **Code Interpreter** if you want it to parse uploaded HAR / PSI JSON files
8. Save → choose visibility (Only me / Anyone with link / Public)

## How to invoke

Open your GPT, paste:

```
I'm about to deploy these changes: [paste diff].
Lighthouse mobile: [paste]. CrUX: [paste]. Audit.
```

Or upload files and ask:

```
Audit page performance of the uploaded HTML/CSS/JS against the attached PSI report.
```

## Limitations vs Claude Code

- **~8000 char Instructions limit** — methodology trimmed to fit. Full version is the linked SKILL.md.
- **No file system access** unless Code Interpreter is enabled.
- **No hooks, no Task tool, no PSI runner**.
- **OpenAI-only** — requires ChatGPT Plus/Team/Enterprise.

## Updating

Edit your GPT in the Builder and replace the Instructions with the latest [`instructions.md`](https://raw.githubusercontent.com/brunnocarpena/performance-audit-skill/main/adapters/chatgpt-custom-gpt/instructions.md).

## Reference

- GPT Builder: https://help.openai.com/en/articles/8554407-gpts-faq
- 8000 char limit: https://community.openai.com/t/how-can-i-increase-the-ceiling-for-gpt-instructions-beyond-8000-characters/801867
