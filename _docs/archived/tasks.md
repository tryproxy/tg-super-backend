# Prototype task backlog

Temporary task draft derived from [plan.md](plan.md). Each task is intended to fit one implementation session. After review and transfer, GitHub Issues become the active backlog. `P-*` references point to the Prototype criteria in the plan.

## T01. Bootstrap the Worker with a passing test

### Goal

Run a minimal TypeScript Cloudflare Worker locally with the approved tooling and one meaningful passing HTTP test.

### Acceptance criteria

- [ ] pnpm scripts install, typecheck, test, and run the Worker locally with Wrangler.
- [ ] A Hono health endpoint responds successfully and has a Vitest test.
- [ ] Worker configuration includes a D1 binding without requiring a running Node.js server in deployment.

### Out of scope

Telegram webhooks, product configuration, and customer messages.

### Constraints

Use the approved stack in `_docs/tech-stack.md`; keep test and demo deployment resources separate.

## T02. Persist product, channel, group, conversation, and delivery identities

### Goal

Add D1 migrations and storage operations for the identities and delivery state needed by the message bridge.

### Acceptance criteria

- [ ] Products, customer channels, groups, conversations, and active topic mappings have explicit keys; one group cannot be active for two products.
- [ ] Conversation identity includes product, channel, and external customer chat; reverse lookup includes group and topic.
- [ ] Processed update IDs and outbound delivery attempts can be stored with uniqueness and timestamps.
- [ ] Migration and storage tests cover duplicate mappings and two products with the same external customer chat ID.

### Out of scope

Admin HTTP endpoints, webhook handlers, and Telegram sends.

### Constraints

Follow P-04, P-09, P-12, P-18, and P-19. Do not store plaintext bot tokens.

## T03. Register dedicated product bots and support groups

### Goal

Expose protected admin endpoints to register and inspect multiple product integrations.

### Acceptance criteria

- [ ] An authorized operator can register a product, its dedicated customer bot, and an existing private forum group.
- [ ] Duplicate active group registration is rejected; read endpoints omit credentials.
- [ ] Product bot tokens are encrypted before storage using a key held in a Worker secret.
- [ ] Unauthorized requests and invalid identifiers are rejected without logging secrets.

### Out of scope

Shell UI, Business accounts, CSV onboarding, and automatic group creation.

### Constraints

Follow P-01, P-02, P-04, and P-06; use the shared service bot only for internal groups.

## T04. Validate and activate Telegram setup

### Goal

Make activation confirm that the product bot, forum group, and service bot are usable before accepting traffic.

### Acceptance criteria

- [ ] Activation checks product bot identity, forum capability, and the service bot's required group access and topic permissions.
- [ ] A bot with another active webhook is not silently taken over.
- [ ] Invalid token, non-forum group, or missing permission leaves the integration inactive with an actionable admin status.
- [ ] Activation configures and records the correct webhook identity and secret for the dedicated bot.

### Out of scope

Manager invitations or granting Telegram permissions automatically.

### Constraints

Follow P-02, P-03, P-05, and P-06. Managers join manually in Prototype.

## T05. Receive and deduplicate Telegram webhooks

### Goal

Accept updates for configured product bots and the service bot, validating their origin and routing identity.

### Acceptance criteria

- [ ] Webhook handlers validate the Telegram secret and receiving bot identity before processing.
- [ ] Private customer messages resolve to the correct product/channel; group updates resolve to the configured service bot and group.
- [ ] Repeated update IDs within seven days do not produce another processing attempt.
- [ ] Unmapped groups, unsupported update kinds, and the service bot's own messages cannot reach a customer.

### Out of scope

Creating topics or delivering customer and manager messages.

### Constraints

Follow P-07, P-12, and P-15; use the D1 identity model from T02.

## T06. Create and recover customer topics for inbound text

### Goal

Map each customer conversation to one active topic and place incoming text in the correct product group.

### Acceptance criteria

- [ ] A first message creates a topic and stores its mapping; later messages reuse it, including concurrent or repeated updates.
- [ ] Topic title uses the current display name or `Client <chat ID>`; later name changes rename the existing topic.
- [ ] A closed topic reopens; a deleted topic is replaced and the mapping is updated.
- [ ] Two products or channels do not share a customer topic, even for the same Telegram chat ID.

### Out of scope

Media, unsupported-content notices, and manager replies.

### Constraints

Follow P-07 through P-09 and the accepted topic-recovery assumption. The first message may be unsupported; T08 adds its notice.

## T07. Deliver manager text replies through the product bot

### Goal

Send ordinary messages from mapped customer topics to the customer through the corresponding product bot.

### Acceptance criteria

- [ ] A participant's text in a mapped topic is sent immediately through the product bot for that conversation.
- [ ] Another group, an unmapped topic, and service-bot output never send a customer reply.
- [ ] A confirmed send failure produces a notice in the same topic without a reply loop.
- [ ] Tests prove that replies for two products use their respective customer bots.

### Out of scope

Media replies, internal notes inside a customer topic, and Telegram Business.

### Constraints

Follow P-13 through P-17, including P-14 customer-visible bot identity. T09 adds durable attempt status and unknown-outcome handling.

## T08. Relay supported media and report unsupported content

### Goal

Relay supported content in both directions and make unsupported or oversized content visible without a false success claim.

### Acceptance criteria

- [ ] Text, photos, documents, and voice messages relay with captions where applicable.
- [ ] Media moved between distinct bots is downloaded and uploaded; a file above the cloud Bot API download ceiling is rejected visibly.
- [ ] Supported forwarded content is treated as ordinary content, without forwarding metadata or reply linkage.
- [ ] An unsupported first customer message creates a topic, shows why it was not relayed, and notifies the customer; unsupported manager content gets a topic notice.

### Out of scope

Albums, edits, stickers, videos, and preserving reply or forwarding context.

### Constraints

Follow P-08, P-10, and P-11; never echo service-bot notices to customers.

## T09. Surface unknown delivery outcomes for manual reconciliation

### Goal

Record each outbound attempt before contacting Telegram and let an operator find outcomes left uncertain by interruption.

### Acceptance criteria

- [ ] An in-progress attempt is persisted before a Telegram send; success and confirmed failure update its status.
- [ ] An unfinished attempt is visible as unknown through the protected admin API and is not resent automatically.
- [ ] The operator can record a manually verified outcome without causing a second send.
- [ ] Tests cover interruption after Telegram accepted a message but before D1 recorded success, as well as a confirmed Telegram failure.

### Out of scope

Exactly-once external delivery, automatic replay of unknown attempts, and a queue service.

### Constraints

Follow P-12, P-16, and P-18. Known failures appear in the topic; unknown outcomes remain distinguishable from failure.

## T10. Clean up bounded operational history

### Goal

Expire short-lived operational records while retaining conversation mappings for active integrations.

### Acceptance criteria

- [ ] Scheduled Worker cleanup removes processed update IDs older than seven days and delivery statuses older than 30 days.
- [ ] Cleanup leaves active product, channel, group, conversation, and topic mappings intact.
- [ ] Tests prove boundary ages and that cleanup preserves customer-to-topic mappings until their integration is deleted.

### Out of scope

A customer message archive or automatic deletion of active integrations.

### Constraints

Follow P-19 and use only the approved Worker/D1 stack.

## T11. Verify two isolated products in test and demo environments

### Goal

Demonstrate the complete Prototype conversation with two products and independent test and demo resources.

### Acceptance criteria

- [ ] Test and demo use separate product bots, service bots, forum groups, webhook endpoints, and D1 databases; local development uses test resources.
- [ ] Two customer bots create isolated topics; subsequent messages and manager replies reach only the corresponding customers.
- [ ] Smoke checks cover supported media, an unsupported first message, closed/deleted topic recovery, invalid setup, repeated updates, confirmed failure, and an unknown delivery outcome.
- [ ] Deployment instructions identify required secrets and verification commands without publishing secret values.

### Out of scope

Production deployment, Shell UI, Telegram Business, CSV onboarding, and MTProto provisioning.

### Constraints

Complete the Prototype completion check in `_docs/archived/plan.md`; stay within Cloudflare Workers Free and D1 Free limits.
