# Telegram Support Backend Concept

Status: Draft

## Purpose

`tg-super-backend` connects a product's customer-facing Telegram endpoint to a private Telegram forum supergroup used by support managers.

Each product may be represented by a remote in the Runtime MF Shell. The Shell is the control plane for configuring the product and observing its Telegram integration. This repository owns Telegram webhooks, routing, persistence, and provisioning integrations.

## Core model

| Term | Meaning |
| --- | --- |
| Product | A support-owning product, currently expected to correspond to a Shell remote. |
| Customer channel | The Telegram endpoint a customer contacts: initially a regular bot, potentially a connected Business account later. |
| Support group | A private forum-enabled supergroup used by a product's managers. |
| Owner account | A Telegram user account that owns or provisions a support group. It is not automatically the customer-facing identity. |
| Manager | A Telegram user allowed to work with customer topics in a product's support group. |
| Conversation | The support relationship between one customer and one product customer channel. |
| Customer topic | The forum topic where managers see and answer one conversation. |
| Connector bot | A platform-controlled bot that may be connected to multiple Telegram Business accounts through separate business connections. |

## Proposed relationship

```text
Product
├── one active customer channel
│   ├── regular Telegram bot, or
│   └── connected Telegram Business account
├── one support forum supergroup
├── one owner account for group governance/provisioning
└── many managers

Conversation
├── belongs to one product/customer channel
├── identifies one external customer chat
└── maps to one topic in the product's support group
```

The owner account, customer channel, and manager accounts are separate roles. The same Telegram account may fill more than one role, but the system must not require that coupling.

## Regular bot flow

1. A customer opens the product's help bot and sends a message.
2. Telegram sends a webhook update to this backend.
3. The backend resolves the product from the receiving bot.
4. The backend finds the conversation by product, channel, and external customer chat.
5. If no topic exists, the backend creates a topic named for the customer and persists the mapping.
6. The backend copies the customer message into the topic.
7. A manager writes a reply in the topic.
8. The backend resolves the conversation from the group and topic and sends the reply to the customer through the product bot.

The customer sees the bot as the sender. Managers work entirely inside the Telegram group.

## Telegram Business flow

The conversation and topic workflow remains the same. Only the customer-channel adapter changes:

1. A product owner connects a Business-enabled connector bot to the product's Telegram account and grants explicit rights.
2. The backend stores the resulting business connection and associates it with the product.
3. Incoming `business_message` updates are normalized into the same internal message shape as ordinary bot messages.
4. Manager replies are sent with the business connection, so the customer sees the Business account as the sender.

A Business connection does not provision or transfer ownership of the support supergroup. Group provisioning remains a separate concern.

## Group provisioning

The Telegram Bot API can manage topics after a bot has the required administrator rights, but it cannot create a supergroup on behalf of a user account.

Two provisioning modes are expected:

- Manual MVP: the owner creates or selects a forum supergroup, adds the support bot, grants required rights, and registers the group in the Shell.
- Automated later: an isolated MTProto provisioning component uses an explicitly authorized user session to create/configure the group and add the bot and managers.

User sessions are high-value credentials and must not be exposed to the Shell browser or stored as ordinary application data.

## Manager onboarding

Managers may be imported from CSV as expected members of a product team. Email is an organizational identifier, not a Telegram identifier.

The reliable onboarding flow is:

1. Import manager name, email, optional Telegram username/phone, and requested role.
2. Create a personal, expiring invite link or join request.
3. Send or display the link through the chosen organizational channel.
4. When the manager joins, capture the Telegram user ID and bind it to the imported manager record.
5. Apply the minimum required membership or administrator privileges.

Direct MTProto invitations may be attempted later, but privacy settings and Telegram limits mean they cannot be guaranteed.

## Architectural boundary

Normalize channel-specific Telegram updates before entering the support workflow:

```text
Telegram adapter
  -> normalized inbound message
  -> conversation/topic router
  -> delivery record
  -> Telegram adapter
```

This keeps ordinary bots and Business accounts interchangeable without duplicating conversation rules.

At minimum, conversation identity must include:

```text
(product_id, customer_channel_id, external_customer_chat_id)
```

Group-side routing must include both group and topic identifiers.

## MVP boundary

The proposed MVP contains:

- a regular Telegram bot customer channel;
- an existing or manually created forum supergroup;
- one customer topic per product/customer conversation;
- bidirectional text-message routing;
- manager onboarding through invite links;
- Shell-visible integration status;
- idempotent webhook handling and persisted mappings.

Telegram Business, automatic group creation, direct manager invitation, rich media parity, internal notes, queues, and advanced lifecycle automation remain follow-up capabilities until explicitly selected.
