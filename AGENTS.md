# tg-super-backend

Backend for routing Telegram customer-support conversations between a product's customer-facing channel and a private forum supergroup.

Use CodeGraph before text search when locating code symbols or call paths if a `.codegraph/` directory exists.

## Required context

- GitHub Issues are the active backlog. Read `_docs/process.md` before working an issue.
- Follow the role cycle in `_docs/process.md`. The main session selects the Issue, delegates grooming to `pm`, and selects the Engineer or QA project agent profile according to `_docs/routing.md`; it does not groom, implement, or verify the Issue itself.
- PM reads the related User Story, SPEC requirements, Decisions, and Open Questions while grooming. Engineer reads the complete groomed Issue and its linked implementation context. QA verifies the acceptance criteria in the Issue and does not reconcile requirements.
- When an Issue references a research Finding, read its entry in `_docs/research/findings.md`. Findings supply evidence and ordering; they do not override accepted requirements.
- Read `_docs/tech-stack.md` before adding a runtime, framework, database, deployment target, or package.
- PM reconciles an Issue with active requirements before implementation. Engineer reports a concrete obstacle instead of changing the contract; QA returns only `PASS` or `FAIL` against the acceptance criteria.
- Read `_docs/archived/` only when linked or needed for missing context. Archived material is historical reference, not an active requirement.

## Current direction

- The named stages are Prototype, MVP, and Release, in that order. Telegram Business, Shell integration, and CSV onboarding belong to MVP. MTProto group provisioning belongs to Release; manager onboarding uses the MVP invitation-link flow.
- Prototype uses one dedicated Telegram support bot per product, manually prepared product forum groups and managers, one common internal service bot, and a protected admin API. Products exist independently of this backend; it stores only their references and Telegram integrations.
- Use TypeScript, Node.js and pnpm for local tooling; deploy webhook handlers on Cloudflare Workers Free with D1 Free.
- Keep the message-processing path serverless: Telegram webhook, conversation/topic lookup, persistence, Telegram delivery.
- Keep Telegram credentials and webhook processing in this backend. The Shell is a later control plane.
- Keep any future MTProto user-account automation outside the normal message-routing path.

## Rules

- Never commit or log bot tokens, Telegram user sessions, login codes, two-factor secrets, webhook secrets, or customer message contents used as fixtures.
- Keep product identity, customer-channel identity, customer chat identity, group identity, and topic identity explicit.
- Make webhook handling idempotent; Telegram updates can be repeated.
- Treat an ordinary message in a mapped customer topic as an immediate external reply. Keep internal discussion outside mapped customer topics until an explicit internal-note feature is decided.
- Give managers the least Telegram privileges required for their role.
- Do not treat an email address as a Telegram identity.
- Prefer the smallest implementation that satisfies the accepted issue. Add queues, Durable Objects, or provisioning infrastructure only when a concrete requirement justifies them.
- When accepted behavior changes, update `SPEC.md` and affected issues. Update User Stories if user goals or their short criteria change, and Decisions if an accepted choice changes.
- Declare dependencies in `package.json` and manage them with pnpm. Add a dependency only when the groomed issue requires it and it fits `_docs/tech-stack.md`; obtain approval and update the tech stack before introducing anything outside it.
- Prefer a focused automated test with each implementation issue. Routing, idempotency, authorization, and conversation-to-topic mapping require focused tests; use an observable check only when automation is not practical.

## Commands

No application toolchain has been initialized yet. Once it exists, record canonical install, development, validation, migration, and deployment commands here and in README.md.
