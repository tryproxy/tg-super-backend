# Proposed Technical Stack

Status: Proposed. This file records the current direction, not an irreversible commitment.

## Recommended MVP stack

| Concern | Choice | Reason |
| --- | --- | --- |
| Language | TypeScript | Fits the existing team experience and Telegram/Cloudflare ecosystem. |
| Runtime/tooling | Node.js 22 and pnpm 11 | Familiar local tooling; avoids requiring Bun for the first implementation. |
| HTTP framework | Hono | Small webhook/API surface, strong Cloudflare Workers support, and portable request handlers. |
| Telegram Bot API | grammY | Typed update handling, middleware, webhook integration, and existing team familiarity. |
| Compute | Cloudflare Workers | Serverless webhook execution with no continuously running bot process. |
| Database | Cloudflare D1 | Managed serverless SQL with SQLite semantics, suitable for product/channel/group/conversation/topic mappings. |
| Validation | Zod | Runtime validation for Shell APIs, CSV rows, configuration, and normalized events. |
| Deployment/config | Wrangler | Canonical local and deployed Cloudflare configuration. |
| Tests | Vitest plus focused integration tests | Fast TypeScript tests for routing, authorization, idempotency, and Telegram adapter behavior. |

The runtime should use Telegram webhooks rather than long polling so Workers only run when requests arrive.

## Logical modules

```text
src/
├── core/                 domain types and errors
├── application/          conversation and provisioning workflows
├── telegram/
│   ├── bot/              ordinary Bot API adapter
│   ├── business/         connected Business adapter
│   └── provisioning/     MTProto boundary, if selected
├── storage/              D1 repositories and migrations
├── http/                 Hono routes and webhook verification
└── worker.ts             Cloudflare entry point
```

This is a responsibility map, not a requirement to create empty directories before implementation needs them.

## Core persistence

The first schema is expected to represent:

- products;
- customer channels and credential references;
- support groups;
- product managers and Telegram identities;
- conversations and topic mappings;
- processed Telegram updates for idempotency;
- message/delivery links when replies, edits, or audit history require them;
- Business connections when that adapter is implemented.

Important uniqueness boundaries include:

```text
(product_id, customer_channel_id, external_customer_chat_id)
(support_group_id, topic_id)
(telegram_bot_id, update_id)
```

## Secrets

- Static deployment secrets belong in Cloudflare secret bindings.
- Dynamically onboarded bot tokens require encrypted storage or a dedicated secret manager; D1 rows must not contain unprotected tokens.
- Telegram user sessions must be isolated from normal application data and from the Shell browser.
- Webhooks must validate Telegram's webhook secret in addition to resolving the receiving bot/connection.

## Reliability path

Start with the smallest reliable path:

```text
Telegram webhook -> Worker -> D1 mapping -> Telegram Bot API
```

Required from the beginning:

- idempotency for repeated webhook updates;
- unique conversation/topic mappings;
- explicit delivery status;
- actionable logging without message or secret leakage.

Add Cloudflare Queues when delivery should be decoupled from webhook latency or requires managed retries and a dead-letter path.

Add a Durable Object keyed by the conversation identity, or another serialization mechanism, when concurrent first messages can create duplicate external topics despite database uniqueness. Do not add it only as a precaution before the risk is demonstrated or required.

## MTProto provisioning

grammY uses the Bot API and cannot create a supergroup on behalf of a user account. If automatic group creation or direct user invitation is selected, implement it through a separate MTProto adapter or service.

The preferred boundary is a short-lived provisioning job with explicit inputs and persisted status. A continuously connected userbot must not become a dependency of ordinary customer-message routing unless the product explicitly requires user-account messaging.

The specific MTProto library and hosting target remain open until automated provisioning becomes an accepted story.

## PostgreSQL alternative

Use serverless PostgreSQL instead of D1 if this backend must share relational data with other services, requires unsupported SQL behavior, or adopts an existing PostgreSQL operational standard. Cloudflare Workers can reach a serverless provider directly or a traditional database through Hyperdrive.

Do not run D1 and PostgreSQL as competing sources of truth for the same conversation mappings.

## Deferred choices

- Prisma or another ORM: decide after D1 versus PostgreSQL is confirmed.
- Cloudflare Queues: add when retry requirements are confirmed.
- Durable Objects: add when per-conversation coordination is required.
- MTProto library and host: add with automated provisioning.
- Email provider: add only if this service is responsible for distributing manager invite links.
