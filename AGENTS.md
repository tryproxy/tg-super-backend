# tg-super-backend

Use CodeGraph before text search when locating code symbols or call paths if a `.codegraph/` directory exists.

The main session selects Issues and delegates PM, Engineer, and QA work using [_docs/process.md](_docs/process.md); it does not perform their work. Each role follows its own file in `_docs/team/`.

Referenced documents are read only when the current task requires them. Archived material is historical context.

## Rules

- Never commit or log bot tokens, Telegram user sessions, login codes, two-factor secrets, webhook secrets, or customer message contents used as fixtures.
- Keep product identity, customer-channel identity, customer chat identity, group identity, and topic identity explicit.
- Make webhook handling idempotent; Telegram updates can be repeated.
- Treat an ordinary message in a mapped customer topic as an immediate external reply. Keep internal discussion outside mapped customer topics until an explicit internal-note feature is decided.
- Give managers the least Telegram privileges required for their role.
- Do not treat an email address as a Telegram identity.
- Prefer the smallest implementation that satisfies the accepted issue. Add queues, Durable Objects, or provisioning infrastructure only when a concrete requirement justifies them.
- Declare dependencies in `package.json` and manage them with pnpm. Add a dependency only when the groomed issue requires it and it fits `_docs/tech-stack.md`; obtain approval and update the tech stack before introducing anything outside it.
- Prefer a focused automated test with each implementation issue. Routing, idempotency, authorization, and conversation-to-topic mapping require focused tests; use an observable check only when automation is not practical.
- Keep the message-processing path serverless.
- Keep Telegram credentials and webhook processing in this backend.
- Keep MTProto account automation outside the message-routing path.

## Commands

Use the scripts in `package.json` as the canonical executable commands. README documents setup and deployment usage when implemented.
