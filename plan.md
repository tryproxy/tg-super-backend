# Implementation Plan

## Goal

Build a serverless backend that routes Telegram customer conversations into product support forum groups and sends manager replies back through the same customer-facing channel.

The backend is the Telegram integration layer. The Runtime MF Shell is a later control plane for configuration and status; it is not part of the first backend implementation.

## First release boundary

The first release should establish the complete conversation path with manual setup:

- support multiple products;
- support a regular product-owned Telegram bot;
- support a Telegram Business account through a connected platform connector bot;
- use one private forum-enabled supergroup for each product;
- use one persistent topic for each (product, customer channel, external customer chat) conversation;
- use one common platform service bot as the internal bridge and group administrator;
- provide a protected admin API for registering products, channels, groups, and manager access;
- add support groups and managers manually;
- route inbound customer messages to the matching topic;
- route ordinary manager replies from that topic back to the customer;
- make webhook processing idempotent and keep delivery status visible.

The owner account that governs a support group, the customer-facing bot or Business account, and manager accounts are separate roles. The design must not require them to be the same Telegram account.

## Core message flow

### Setup

1. Register a product through the admin API.
2. Register the product's regular bot or Business connection.
3. Create or select the product's forum supergroup manually.
4. Add the common service bot to the group with the minimum rights required to read messages and manage topics.
5. Register the group and its Telegram identifiers.
6. Add managers to the group manually and grant their configured Telegram permissions.

### Customer to support

1. Telegram sends a webhook update to the backend.
2. The adapter resolves the product and customer channel.
3. The backend normalizes the update into a common inbound message.
4. It finds the conversation by product, customer channel, and external customer chat.
5. If the conversation has no topic, it creates one and stores the mapping.
6. It copies the customer message into that topic.
7. It records the delivery result without logging tokens or message contents.

### Support to customer

1. The service bot receives a message from a mapped topic in a product group.
2. The router resolves the conversation from the group and topic identifiers.
3. It sends the manager message through the configured channel:
   - regular bot: Bot API sendMessage or the matching media method;
   - Business channel: the connected Business context and business_connection_id.
4. It records success or an actionable failure in the topic and delivery data.
5. Every ordinary manager message in a customer topic is treated as a customer reply in the first release. Internal discussion must happen outside customer topics until an explicit internal-note flow is designed.

## Logical components

- **Admin API**: protected product, channel, group, manager, and status operations. It can be called directly during the manual phase and later by the Shell.
- **Telegram webhook layer**: validates webhook secrets and dispatches updates to the correct product/channel adapter.
- **Channel adapters**: keep regular Bot API updates and Telegram Business updates behind one normalized interface.
- **Conversation and topic router**: owns conversation identity, topic creation/reuse, authorization, and delivery decisions.
- **Persistence**: stores products, channel references, group/topic mappings, manager Telegram identities, business connections, processed updates, and delivery state.
- **Service bot integration**: one platform-controlled bot is used as the internal bridge and can administer all configured product support groups.
- **Provisioning boundary**: reserved for later MTProto automation; it must not be part of ordinary message routing.

## Technical direction

Use the proposed serverless stack:

- TypeScript;
- Node.js 22 and pnpm;
- Hono for the HTTP and admin API;
- grammY for Telegram Bot API integration;
- Cloudflare Workers and webhooks;
- Cloudflare D1 for the initial relational state;
- Zod for API, configuration, and normalized update validation;
- Wrangler for deployment;
- Vitest for focused routing and integration tests.

Keep secrets outside ordinary API responses and logs. Bot tokens require protected secret storage. User-account sessions, if MTProto is added later, require a separate protected boundary and must never be exposed to the Shell browser.

## Delivery phases

### Phase 1 — Manual backend MVP

- Define the domain model and migrations.
- Implement protected admin endpoints.
- Register products, regular bots, Business connections, support groups, and managers.
- Implement webhook verification and idempotency.
- Implement regular Bot API inbound/outbound routing.
- Implement Business update normalization and outbound delivery.
- Implement topic creation/reuse and group/topic authorization.
- Add focused tests for identity boundaries, duplicate updates, topic mapping, permissions, and delivery failures.
- Deploy a minimal Worker and verify the complete manual setup path.

### Phase 2 — Shell integration

- Expose the backend configuration and health model to the Runtime MF Shell.
- Add Shell operations for product/channel/group setup and connection status.
- Show actionable states for invalid credentials, missing group permissions, disconnected Business accounts, webhook problems, and delivery failures.
- Keep Telegram credentials and account sessions backend-only.

### Phase 3 — MTProto provisioning

- Add an isolated provisioning component for explicitly authorized Telegram user sessions.
- Automate creation and forum configuration of a support supergroup when selected.
- Add the service bot with required rights.
- Make provisioning resumable and report status without exposing session secrets.
- Keep provisioning failures separate from message-routing failures.

### Phase 4 — CSV manager onboarding

- Define and validate the CSV contract.
- Import manager records without treating email as Telegram identity.
- Generate controlled personal invite links or join requests.
- Bind a manager record to the Telegram user ID after the manager joins.
- Apply the minimum configured role permissions.
- Add direct MTProto invitations only if they remain necessary after the invite-link flow.

## Verification gates

Before considering the first release complete, verify:

- two products cannot route into each other's groups or topics;
- the same customer can use both channels without conversation-key collisions;
- repeated Telegram updates do not create duplicate topics or messages;
- only a mapped support-group topic can send a manager reply externally;
- Business messages use the correct business_connection_id;
- a missing permission, closed topic, blocked customer, or failed delivery is visible and recoverable;
- tokens, user sessions, and customer message contents do not appear in logs or API responses;
- existing _docs/* and AGENTS.md remain unchanged by this planning change.
