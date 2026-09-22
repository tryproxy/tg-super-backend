# Product Specification

Status: living product specification. The current scope is Prototype; MVP and Release describe accepted direction and remain subject to their open questions.

This document is the canonical source for observable product behavior. User Stories express user goals and short acceptance criteria. Decisions record why important choices were made. GitHub Issues split this specification into implementation units and add task-specific verification.

## 1. Product goal

Describe the support flow in which customers write to a product's Telegram support channel, managers work in a forum supergroup, and our backend routes messages between the customer channel and the customer's topic.

## 2. Scope by stage

### Prototype

Regular Telegram bots, manually prepared forum supergroups and managers, one common service bot, protected admin API, and serverless message routing.

### MVP

Telegram Business, Runtime MF Shell integration, CSV manager onboarding, and manager invitations.

### Release

User-account-based MTProto group provisioning and direct manager invitations where Telegram permits them.

## 3. Actors and Telegram identities

### Operator

An operator registers products through the protected administrative API and configures a Telegram support bot and support supergroup for each product during the Prototype.

### Telegram support bot

An ordinary Telegram bot dedicated to the support service of one product. In the Prototype, the bot is the customer-facing channel for that product and is not shared with another product or another backend.

### Common service bot

One Telegram bot used by the platform in every product support supergroup. During the Prototype, an operator adds it to each group manually. It receives manager messages and creates or manages customer topics. It is separate from the Telegram support bots that customers contact.

### Manager

A member of a product support supergroup who works with customer conversations in its topics. During the Prototype, managers are added to the group manually; the backend does not invite them or assign their Telegram permissions.

### Customer

A person who contacts a product through its Telegram support channel. In the Prototype, the customer writes privately to the product's Telegram support bot.

The responsibilities and permissions of the remaining actors and Telegram identities will be defined as their User Stories are groomed:

- product owner;
- shared Telegram Business connector bot;
- product owner's user account used for Release provisioning.

## 4. Domain model and vocabulary

### Product

A separate support unit. Registering a product adds its reference and Telegram support settings to this backend. In the MVP, the reference may identify a product or remote in Runtime MF Shell; the authoritative Shell identifier remains an open question.

### Telegram integration

The Telegram support settings registered for one product. In the Prototype, they contain one dedicated Telegram support bot and one existing private forum supergroup with topics enabled.

### Support supergroup

The product's active private Telegram forum supergroup. One active supergroup cannot be assigned to two products at the same time.

### Customer channel

The Telegram identity through which a customer contacts a product. In the Prototype, this is the product's dedicated Telegram support bot. Telegram Business adds another channel type in the MVP.

### External customer chat

The Telegram private chat from which the customer contacts a particular customer channel. Its Telegram chat ID is used as part of the conversation identity.

### Conversation

The continuing support exchange identified by product, customer channel, and external customer chat. The same Telegram user contacting another product or another channel starts a separate conversation.

### Customer topic

The forum topic associated with one conversation. A conversation has one current topic: a closed topic is reopened, while a deleted topic is replaced and the stored association is updated.

The remaining terms will be defined as their User Stories are groomed:

- Business connection;
- delivery attempt;

Record the relationships and invariants between these terms here before implementation introduces storage entities.

## 5. System flows

### Prototype: customer message to support topic

1. Telegram sends the customer's update to the webhook of the Telegram support bot.
2. The receiving bot identifies the product and customer channel.
3. The backend finds the conversation from the product, customer channel, and external Telegram chat.
4. The backend reuses the conversation's topic, reopens it if closed, or creates and stores a replacement if it was deleted.
5. The common service bot places the customer's message or an unsupported-content notice in that topic.

### Prototype: manager reply to customer

1. A manager sends an ordinary message in a customer topic.
2. The common service bot receives the group update and resolves the conversation from the support supergroup and topic.
3. The backend rejects messages from another group, an unknown topic, or the common service bot itself.
4. The backend stores an in-progress delivery attempt before calling Telegram.
5. The same Telegram support bot originally contacted by the customer sends the message to the customer's private chat.
6. The backend records success or a confirmed failure. If the result is unknown, it keeps that state and posts a warning in the customer topic without retrying automatically.

The remaining normative diagrams and flows will cover:

1. MVP Telegram Business connection and message routing.
2. Release user-account authorization and support-group provisioning.

## 6. Functional requirements

Convert the accepted behavior from User Stories into numbered requirements. Each requirement should identify its stage and link back to the relevant User Story, for example `PRO-01` and `MVP-01`.

### 6.1 Setup and access

- `PRO-01` (US-01): An operator can register multiple products through the protected administrative API and configure Telegram support for each of them.
- `PRO-02` (US-01): Telegram support settings belong to exactly one registered product.
- `PRO-03` (US-01): Each registered product has one dedicated Telegram support bot and one existing private forum supergroup with topics enabled.
- `PRO-04` (US-01): The support bot is dedicated to this service. Preserving another backend or webhook for that bot is not required.
- `PRO-05` (US-01): One active support supergroup cannot be connected to two products at the same time.
- `PRO-06` (US-01): An integration remains inactive until its configuration passes validation. The concrete checks are refined by US-02 and US-06.
- `PRO-07` (US-01): Administrative read operations do not return bot tokens or other Telegram secrets, and secrets are not written to logs.
- `PRO-08` (US-02): The same common service bot is used in every product support supergroup and remains separate from the Telegram support bots used by customers.
- `PRO-09` (US-02): During the Prototype, an operator manually adds the common service bot and managers to the product support supergroup.
- `PRO-10` (US-02): The common service bot must be an administrator of the support supergroup and must be able to receive group messages and create or manage forum topics. The integration remains inactive if the bot is missing or lacks the required access.
- `PRO-11` (US-02): A failed group or permission check identifies what the operator must correct without exposing Telegram secrets.
- `PRO-12` (US-02): Manager invitations, identity matching, and automatic assignment of Telegram permissions are outside the Prototype.
- `PRO-38` (US-06): A protected administrative API exposes whether a product's Telegram integration is ready to receive and route messages without requiring a Shell interface.
- `PRO-39` (US-06): Readiness checks verify that the configured Telegram support bot is accessible and has the expected bot identity.
- `PRO-40` (US-06): Readiness checks verify that the Telegram support bot's webhook points to the expected backend endpoint and uses the expected webhook protection.
- `PRO-41` (US-06): Readiness checks verify that the configured support supergroup exists and has forum topics enabled.
- `PRO-42` (US-06): Readiness checks verify that the common service bot belongs to the support supergroup and has the access required to receive messages and manage topics.
- `PRO-43` (US-06): Each readiness check returns its own result and an actionable reason when it fails, so the operator can identify what must be corrected.
- `PRO-44` (US-06): The readiness response is limited to setup checks. It does not expose customer message history or present delivery history as setup status.
- `PRO-45` (US-06): Administrative responses never return bot tokens, webhook secrets, other Telegram credentials, or customer message contents.

### 6.2 Customer messages and topic lifecycle

- `PRO-13` (US-03): A private customer message received by a configured Telegram support bot resolves to that bot's product and customer channel.
- `PRO-14` (US-03): A conversation is identified by product, customer channel, and external Telegram chat. Changing any part of that identity produces a different conversation.
- `PRO-15` (US-03): The first message creates and stores no more than one customer topic for the conversation, including when the first message has an unsupported format or concurrent processing occurs.
- `PRO-16` (US-03): Later messages from the same conversation use its current customer topic.
- `PRO-17` (US-03): A closed customer topic is reopened. If Telegram reports that the topic was deleted, the backend creates a replacement, updates the stored association, and uses the replacement for the current and later messages.
- `PRO-18` (US-03): A customer topic uses the customer's current Telegram display name when available and falls back to `Client <chat ID>`. A later name change updates the existing topic title without creating another conversation.
- `PRO-19` (US-03): Repeated webhook updates do not create another topic or repeat the customer's message. The deduplication key includes the receiving bot and Telegram update ID.
- `PRO-20` (US-03): Processed update IDs are retained for seven days and may be removed after that period.

### 6.3 Manager replies and delivery outcomes

- `PRO-28` (US-05): An ordinary message sent by a manager in a customer topic is treated as an immediate reply to that customer.
- `PRO-29` (US-05): Only messages from the configured support supergroup and a topic associated with a conversation may be delivered to a customer.
- `PRO-30` (US-05): A manager reply is sent through the same Telegram support bot that the customer originally contacted.
- `PRO-31` (US-05): Messages produced by the common service bot, including delivery notices, are never sent back to the customer.
- `PRO-32` (US-05): Before calling Telegram, the backend stores a delivery attempt with an in-progress status. The attempt is later recorded as successful, failed, or unknown.
- `PRO-33` (US-05): A confirmed delivery failure produces a clear notice in the same customer topic without creating a reply loop.
- `PRO-34` (US-05): If a failure leaves the delivery result unknown, the customer topic warns that the message may already have been delivered. The backend does not retry the attempt automatically.
- `PRO-35` (US-05): If a manager decides to send the message again, the new message creates a separate delivery attempt and does not change the unknown status of the earlier attempt.
- `PRO-36` (US-05): Every ordinary manager message in a customer topic is an external reply. Internal discussion takes place outside customer topics.
- `PRO-37` (US-05): Delivery statuses are retained for 30 days and may be removed after that period.

### 6.4 Media, captions, and unsupported content

- `PRO-21` (US-04): Text messages, photos up to 10 MB, documents up to 20 MB, and voice messages up to 20 MB are transferred between the customer and the customer's topic.
- `PRO-22` (US-04): Captions attached to supported photos and documents are preserved.
- `PRO-23` (US-04): Files received by one Telegram bot are downloaded and uploaded through the destination bot because Telegram `file_id` values cannot be reused by another bot.
- `PRO-24` (US-04): Supported content from a forwarded message is delivered as an ordinary new message without forwarding metadata or a reply link to the source message.
- `PRO-25` (US-04): An unsupported or oversized first customer message still creates the conversation's topic. The topic explains why the content was not transferred, and the customer receives a delivery-failure notice through the Telegram support bot.
- `PRO-26` (US-04): Unsupported or oversized manager content is not sent to the customer. The manager sees the reason in the customer's topic.
- `PRO-27` (US-04): The backend does not report unsupported or oversized content as successfully delivered.

### 6.5 Telegram Business

### 6.6 Shell, CSV, and invitations

### 6.7 MTProto provisioning

## 7. Reliability, security, and data retention

Webhook processing is idempotent within the seven-day processed-update retention period. Topic recovery preserves the conversation when a topic is closed or deleted. Outbound attempts are stored before Telegram is called, unknown outcomes are not retried automatically, and delivery statuses are retained for 30 days. Administrative setup and readiness responses do not expose Telegram secrets or customer messages.

- `PRO-46` (US-03, Open Question 4): Conversation-to-topic associations are retained until the corresponding Telegram integration is deleted.
- `PRO-47` (Open Question 6): Development and demonstration use separate Telegram support bots, common service bots, support supergroups, webhook endpoints, and D1 databases. Local development uses the test resources; production resources are not prepared for the Prototype. Tokens and resource IDs are assigned during deployment.

## 8. Scope boundaries and open questions

List excluded capabilities by stage and link unresolved decisions to [_docs/open-questions.md](_docs/open-questions.md). Resolved assumptions belong in the requirements or in [_docs/decisions.md](_docs/decisions.md), not in this list.

## 9. Traceability

Maintain a table mapping User Stories to specification requirements and GitHub Issues. Update this table before changing an Issue's scope.

| User Story | Specification requirements | Prototype Issues |
| --- | --- | --- |
| US-01 | PRO-01–PRO-07 | #3 and #4, pending reconciliation |
| US-02 | PRO-08–PRO-12 | #4, pending reconciliation |
| US-03 | PRO-13–PRO-20 and PRO-46 | #2, #5, and #6, pending reconciliation |
| US-04 | PRO-21–PRO-27 | #8, pending reconciliation |
| US-05 | PRO-28–PRO-37 | #7, #9, and #10, pending reconciliation |
| US-06 | PRO-38–PRO-45 | #3 and #4, pending reconciliation |
| Cross-cutting Prototype environment | PRO-47 | #11, pending reconciliation |
