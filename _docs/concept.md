# Telegram support concept

The service connects each product's customer-facing Telegram channel to a private forum supergroup used by its support managers.

## Core roles

| Term | Meaning |
| --- | --- |
| Product | A separately configured support destination; it may later map to a Runtime MF Shell remote. |
| Customer channel | The Telegram identity the customer contacts: a product bot in Prototype, with Business support in MVP. |
| Support group | One active private forum supergroup per product. |
| Owner account | The Telegram user account that governs a group; Prototype setup is manual. |
| Service bot | One platform bot that operates in all support groups and bridges topic messages. |
| Manager | A Telegram group participant who answers customers. |
| Conversation | One customer chat with one product through one customer channel. |
| Customer topic | The active mapped forum topic for one conversation; a deleted topic is replaced. |

The owner account, customer-facing identity, and manager accounts can differ.

## Prototype flow

1. An operator registers a product bot and an existing forum group using the protected admin API.
2. The service bot is made a group administrator; managers are added manually.
3. A customer writes to the product bot. The backend finds or creates the conversation topic in that product's group.
4. The backend delivers the customer message into the topic.
5. A manager writes in the mapped topic. The backend immediately relays the message through the product bot.
6. A confirmed delivery failure is reported in the same topic; an unknown outcome is shown to the operator for manual reconciliation.

A conversation key includes product, customer channel, and external customer chat. Telegram webhook handling is idempotent.

## Stages

- **Prototype:** ordinary product bots, manual group and manager setup, admin API, serverless message bridge.
- **MVP:** Telegram Business support through a shared connector bot, Runtime MF Shell integration, CSV manager onboarding and invitations. Bot and Business channels may coexist for one product.
- **After MVP:** possible MTProto group provisioning and direct manager invitations; there is no named third stage.

The accepted product behavior is in [plan.md](../plan.md) and [specification.md](specification.md). Cross-task rules are in [decisions.md](decisions.md). Technical details are in [architecture.md](../architecture.md).
