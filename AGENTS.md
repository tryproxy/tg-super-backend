# tg-super-backend

Backend for routing Telegram customer-support conversations between a product's customer-facing Telegram endpoint and an internal forum supergroup.

Use CodeGraph before text search when locating code symbols or call paths if a `.codegraph/` directory exists.

## Required context

- Read `_docs/concept.md` before changing product behavior or domain boundaries.
- Read `_docs/user-stories.md` before implementing a feature.
- Read `_docs/open-questions.md` before making an assumption that affects Telegram identity, ownership, permissions, or message delivery.
- Read `_docs/tech-stack.md` before introducing a runtime, framework, database, deployment target, or package.
- Treat unresolved questions as open. Record an explicit decision before implementing behavior that depends on one.

## Current technical direction

- Use TypeScript on Node.js with pnpm. Do not introduce Bun unless the stack decision is deliberately revisited.
- Keep the message-processing path serverless-friendly: Telegram webhook, normalized event, conversation routing, persistence, and Telegram response.
- Keep the Runtime MF Shell as a control plane. Telegram tokens, account sessions, webhook handling, and message routing belong in this backend.
- Use the Telegram Bot API for normal support traffic.
- Isolate MTProto user-account automation behind a provisioning boundary. Do not spread user-session handling through the message-routing code.
- Model a customer-facing endpoint behind an adapter so a regular bot and a connected Telegram Business account can share the same conversation and topic workflow.

## Rules

- Never commit or log bot tokens, Telegram user sessions, login codes, two-factor secrets, webhook secrets, or customer message contents used as fixtures.
- Keep product identity, customer identity, group identity, and topic identity explicit. A conversation key must include the product/customer-channel boundary; a Telegram user ID alone is not globally sufficient.
- Make webhook handling idempotent. Assume Telegram or queue delivery can be repeated.
- Give managers the least Telegram privileges required for their role.
- Do not treat an email address as a Telegram identity. Resolve and store the Telegram user ID after the manager joins or is invited.
- Keep customer-facing messages and internal notes distinguishable. Do not silently choose delivery semantics while that question remains open.
- Prefer the smallest implementation that satisfies the accepted user story. Add queues, Durable Objects, or a separate provisioning service when a concrete reliability or concurrency requirement justifies them.
- When behavior changes, update the relevant file under `_docs/` in the same change.
- Add focused tests for routing, idempotency, authorization, and conversation-to-topic mapping once implementation begins.

## Commands

No application toolchain has been initialized yet. Once it exists, keep canonical install, development, validation, migration, and deployment commands here and in `README.md`.
