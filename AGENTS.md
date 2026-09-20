# tg-super-backend

Backend for routing Telegram customer-support conversations between a product's customer-facing channel and a private forum supergroup.

Use CodeGraph before text search when locating code symbols or call paths if a .codegraph/ directory exists.

## Required context

- Read _docs/decisions.md before changing product behavior, architecture, or scope.
- Read plan.md and _docs/specification.md before implementing Prototype behavior.
- Read _docs/user-stories.md for the relevant story and _docs/open-questions.md before assuming an unresolved Telegram rule.
- Read _docs/tech-stack.md and architecture.md before adding a runtime, framework, database, deployment target, or package.
- If a task conflicts with an accepted decision, surface the conflict before implementing it.

## Current direction

- The named stages are Prototype and MVP. Telegram Business, Shell integration, and CSV onboarding belong to MVP.
- Prototype uses product-owned ordinary bots, manually prepared product forum groups and managers, one common internal service bot, and a protected admin API.
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
- Prefer the smallest implementation that satisfies the accepted story. Add queues, Durable Objects, or provisioning infrastructure only when a concrete requirement justifies them.
- When behavior changes, update the relevant specification and decisions in the same change.
- Add focused tests for routing, idempotency, authorization, and conversation-to-topic mapping once implementation begins.

## Commands

No application toolchain has been initialized yet. Once it exists, record canonical install, development, validation, migration, and deployment commands here and in README.md.
