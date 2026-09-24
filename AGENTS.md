# tg-super-backend

Use CodeGraph before text search when locating code symbols or call paths if a `.codegraph/` directory exists.

For the implementation Issue cycle, the main session selects Issues and delegates PM, Engineer, and QA work using [_docs/process.md](_docs/process.md); it does not perform their work. Each role follows its own file in `_docs/team/`. This delegation rule does not apply to other direct user requests, such as documentation edits.

Referenced documents are read only when the current task requires them. Archived material is historical context.

## Rules

- Never commit or log bot tokens, Telegram user sessions, login codes, two-factor secrets, webhook secrets, or customer message contents used as fixtures.
