# Product Specification

Status: living product specification. The current scope is Prototype; MVP and Release describe accepted direction and remain subject to their open questions.

This document is the canonical source for observable product behavior. User Stories express user goals and short acceptance criteria. Decisions record why important choices were made. GitHub Issues split this specification into implementation units and add task-specific verification.

## 1. Product goal

Customers contact a product's Telegram support channel, while managers handle each conversation in a forum supergroup topic. The backend routes messages between the customer channel and that topic.

## 2. Scope by stage

### Prototype

Regular Telegram bots, manually prepared forum supergroups and managers, one common service bot, protected admin API, and serverless message routing.

### MVP

Telegram Business, Runtime MF Shell integration, CSV manager onboarding, and manager invitations.

### Release

User-account-based MTProto provisioning of a product support supergroup.

## 3. Actors and Telegram identities

### Operator

An operator registers products through the protected administrative API and configures a Telegram support bot and support supergroup for each product during the Prototype.

### Telegram support bot

An ordinary Telegram bot dedicated to the support service of one product. In the Prototype, the bot is the customer-facing channel for that product and is not shared with another product or another backend.

### Common service bot

One Telegram bot used by the platform in every product support supergroup. During the Prototype, an operator adds it to each group manually. It receives manager messages and creates or manages customer topics. It is separate from the Telegram support bots that customers contact.

### Manager

A person allowed to write in a product support supergroup's customer topics. In Prototype, MVP, and Release, an ordinary message from such a person in a mapped topic is an external reply; the backend does not require a matching CSV manager record as an additional sending permission. During the Prototype, managers are added manually. In the MVP, an account can also be linked to an imported manager record after the product owner confirms a join request.

### Customer

A person who contacts a product through its Telegram support channel. In the Prototype, the customer writes privately to the product's Telegram support bot.

### Product owner

The person who configures a product's Telegram support in Shell, connects its Telegram Business account when used, uploads a manager CSV, and confirms managers' join requests. The owner chooses in Telegram which Business chats the connected bot may handle.

### Shared Telegram Business connector bot

One platform bot connected to Business accounts in the MVP. It receives the Business messages Telegram makes available and sends manager replies on behalf of those accounts when the connection allows it. It is separate from the common service bot in support supergroups and from ordinary Telegram support bots.

### Telegram group owner account

The product owner's Telegram user account authorized through a QR code shown in Shell for Release provisioning. The owner approves the login in an already signed-in Telegram app. The separate MTProto component creates the support supergroup under this account, which remains its Telegram owner. This account may differ from the Telegram Business account and is separate from all bots. Authorized-session storage and fallback login remain open questions.

## 4. Domain model and vocabulary

### Product

A separate support unit. Registering a product adds its reference and Telegram support settings to this backend. In the MVP, the reference may identify a product or remote in Runtime MF Shell; the authoritative Shell identifier remains an open question.

### Telegram integration

The Telegram support settings registered for one product. In the Prototype, they contain one dedicated Telegram support bot and one existing private forum supergroup with topics enabled. In the MVP, the supergroup remains required, while the product can use an ordinary support bot, Telegram Business, or both.

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

### Business connection

Telegram's connection between a Business account and the shared connector bot. The backend associates that connection with one product. Telegram determines which private chats are available to the bot from the account owner's settings.

### Delivery attempt

A stored record of one manager reply sent toward a customer. It is created before calling Telegram and records an in-progress, successful, failed, or unknown outcome. An explicit resend is a new attempt.

## 5. System flows

### Prototype: customer message to support topic

1. Telegram sends the customer's update to the webhook of the Telegram support bot.
2. The receiving bot identifies the product and customer channel.
3. The backend finds or creates the conversation identified by the product, customer channel, and external Telegram chat.
4. If a topic is missing or was deleted, the backend reserves its creation for this conversation before asking Telegram to create it. On confirmed success, it stores the topic ID. An existing topic is reused or reopened if closed.
5. If creation has an unknown outcome, the backend does not create another topic automatically. The Telegram support bot tells the customer that transfer to support is unconfirmed, and the operator can review the unresolved conversation through a separate protected operational API.
6. Once the topic is known, the common service bot places the customer's message or an unsupported-content notice in it.

### Prototype: manager reply to customer

1. A manager sends an ordinary message in a customer topic.
2. The common service bot receives the group update and resolves the conversation from the support supergroup and topic.
3. The backend rejects messages from another group, an unknown topic, or any bot.
4. The backend stores an in-progress delivery attempt before calling Telegram.
5. The same Telegram support bot originally contacted by the customer sends the message to the customer's private chat.
6. The backend records success or a confirmed failure. If the result is unknown, it keeps that state and posts a warning in the customer topic without retrying automatically.

### MVP: Telegram Business messages

1. The product owner connects the shared Business connector bot to their Telegram account and chooses which private chats to give it access to in Telegram.
2. Telegram sends messages from those chats to the connected bot. The backend identifies the product from the Business connection.
3. The backend finds or creates a separate customer topic for that product, Business channel, and customer chat, then shows the message to managers there.
4. A manager's reply in that topic is sent to the customer on behalf of the Business account when Telegram permits it.

### MVP: Shell setup and CSV invitation preparation

1. The product owner configures an ordinary support bot, Telegram Business, or both in Shell and specifies the product's existing support supergroup.
2. Shell sends the configuration to the backend API and displays backend-checked connection status and actionable setup errors. Stored Telegram credentials are not returned to Shell after registration.
3. The owner uploads a manager CSV. The backend reports invalid rows and prepares a separate group invitation link for each valid row.
4. A manager follows a prepared link and requests to join. The backend shows the applicant's Telegram account beside the manager record associated with that link; the link alone does not verify the applicant's identity.
5. The product owner confirms the applicant in Shell. The backend asks Telegram to approve the join request. After Telegram confirms success, it links the Telegram ID to the manager record and marks the manager as joined; if approval fails, Shell shows the reason and keeps the applicant unjoined.
6. The CSV column format and delivery of prepared links remain open questions.

### Release: create a support supergroup

1. Shell requests a short-lived QR login token from the separate MTProto component and displays it to the product owner. An expired QR code is refreshed.
2. The owner scans and approves the QR code in an already signed-in Telegram app. The MTProto component confirms the authorized user account; Shell and the operator do not receive an exported user session. Session storage and any fallback login remain open questions.
3. Provisioning checks whether the product already has a support supergroup. If it does, the backend reports the existing group and does not create another.
4. If there is no group, the backend first stores a provisioning operation with a unique marker. The MTProto component creates a private forum supergroup under the authorized user account and places that marker in its description. The user account remains the group owner.
5. On confirmed creation, the backend records the group ID and product association before further setup. If the result is unknown, it does not create another group automatically.
6. To recover an unknown result, the authorized account lists its groups and checks their descriptions for the marker, asking the owner to authorize the account again by QR if necessary. A matching group is associated with the product. If the result still cannot be verified, Shell shows that review is required before another creation attempt.
7. Provisioning adds the common service bot, grants the rights needed for message handling, topics, and MVP manager invitations, then verifies its access. The temporary marker is removed after setup succeeds.
8. If authorization or bot setup fails, Shell shows the cause. A retry continues configuring any recorded group instead of creating a replacement.

## 6. Functional requirements

Requirement prefixes identify their stage: `PRO` for Prototype, `MVP` for MVP, and `REL` for Release. Requirements link to User Stories where applicable; cross-cutting requirements may have no User Story.

### 6.1 Setup and access

- `PRO-01` (US-01): An operator can register multiple products through the protected administrative API and configure Telegram support for each of them.
- `PRO-02` (US-01): Telegram support settings belong to exactly one registered product.
- `PRO-03` (US-01): During the Prototype, each registered product has one dedicated Telegram support bot and one existing private forum supergroup with topics enabled.
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
- `PRO-19` (US-03): While a processed update ID is retained, a repeated webhook update does not create another topic or repeat the customer's message. The deduplication key includes the receiving bot and Telegram update ID.
- `PRO-20` (US-03): Processed update IDs are retained for seven days and may be removed after that period.
- `PRO-48` (US-03): Before requesting a new or replacement customer topic, the backend stores a creation reservation for the conversation so concurrent updates cannot start another creation. An unknown Telegram result keeps that reservation unresolved and is not retried automatically.
- `PRO-49` (US-03): When topic creation cannot be confirmed, the Telegram support bot tells the customer that transfer to support is unconfirmed. A protected operational API, separate from the US-06 setup-readiness response, exposes the unresolved conversation and status without customer message contents or secrets. After checking Telegram, the operator can link a verified existing topic or authorize a new attempt.

### 6.3 Manager replies and delivery outcomes

- `PRO-28` (US-05): An ordinary message sent by a manager in a customer topic is treated as an immediate reply to that customer.
- `PRO-29` (US-05): Only ordinary messages from non-bot members of the configured support supergroup, sent in a topic associated with a conversation, may be delivered to a customer.
- `PRO-30` (US-05): A manager reply is sent through the same Telegram support bot that the customer originally contacted.
- `PRO-31` (US-05): Messages produced by any bot, including delivery notices from the common service bot, are never sent back to the customer.
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
- `PRO-24` (US-04): Supported content from a forwarded message is delivered as an ordinary new message without forwarding metadata. Supported content sent using Telegram's Reply feature is delivered as a new message without a link to the original message; this rule also applies in MVP and Release.
- `PRO-25` (US-04): An unsupported or oversized first customer message still creates the conversation's topic. The topic explains why the content was not transferred, and the customer receives a delivery-failure notice through the Telegram support bot.
- `PRO-26` (US-04): Unsupported or oversized manager content is not sent to the customer. The manager sees the reason in the customer's topic.
- `PRO-27` (US-04): The backend does not report unsupported or oversized content as successfully delivered.

### 6.5 Telegram Business

- `MVP-01` (US-07): A product owner can connect a Telegram Business account to support through the platform's shared Business connector bot. The Business connection is associated with one product through one-time pairing.
- `MVP-02` (US-07): The backend verifies that the Business connection is active and checks the rights Telegram granted to the connector bot. An account already connected to another Business bot cannot be connected without changing that connection in Telegram.
- `MVP-03` (US-07): The product owner chooses in Telegram which private chats the connected bot may handle. Telegram applies that choice; the backend handles the Business messages Telegram delivers and does not fetch or store a separate list of permitted chats.
- `MVP-04` (US-07): Each customer's Business conversation has its own topic in the product's support supergroup. If the product also uses an ordinary Telegram support bot, its conversations use separate topics. The same topic lifecycle, uncertain-creation recovery, and seven-day webhook deduplication rules apply to both channels.
- `MVP-05` (US-07): Manager replies from a Business customer topic are sent on behalf of the connected Business account when Telegram permits the reply.
- `MVP-06` (US-07): Replies sent manually from the Business account appear in the corresponding customer topic as already sent and are not delivered to the customer a second time.

### 6.6 Shell, CSV, and invitations

- `MVP-07` (US-08): A product owner can configure Telegram support for a product in Runtime MF Shell through this backend's API. The product must have at least one customer channel: a dedicated Telegram support bot, Telegram Business, or both.
- `MVP-08` (US-08): Shell lets the product owner specify an existing support forum supergroup; the backend stores its association with the product. MVP does not create the group automatically.
- `MVP-09` (US-08): Shell displays backend-checked status for each configured support channel and the supergroup, with an actionable reason for each failed check. A Business-only product does not require ordinary-bot readiness checks. Shell does not inspect Telegram directly.
- `MVP-10` (US-08): After registration, the backend does not return stored bot tokens or other persistent Telegram credentials to Shell.
- `MVP-11` (US-09): A product owner can upload a CSV list of expected managers for a product. The backend reports which rows are valid and the errors in invalid rows. The required CSV columns remain an open question.
- `MVP-12` (US-09): For each valid manager row, the backend prepares a separate invitation link to the product's existing support supergroup. How those links reach managers remains an open question.
- `MVP-13` (US-09): A CSV email address is not treated as a Telegram identity. Linking a joined Telegram account to an expected manager is covered by US-10.
- `MVP-14` (US-10): Links prepared from the CSV require a Telegram join request; opening a link does not grant membership automatically. The common service bot has the Telegram invitation right needed to create links and handle requests.
- `MVP-15` (US-10): For a join request made through a prepared link, Shell shows the applicant's Telegram account and the associated imported manager record to the product owner. An unrecognized request is not approved automatically.
- `MVP-16` (US-10): After the product owner confirms the applicant, the backend asks Telegram to approve the join request. It associates the applicant's Telegram ID with the manager record only after Telegram confirms approval.
- `MVP-17` (US-10): Shell shows the manager's joining status and role. A failed approval remains visible with its reason and does not mark the manager as joined.
- `MVP-18` (US-10): Joined managers receive only the group permissions needed to work in support topics; the invitation flow does not grant Telegram administrator rights by default.

### 6.7 MTProto provisioning

- `REL-01` (US-11): Shell displays a short-lived QR code generated by the MTProto component. The product owner scans and approves it in an already signed-in Telegram app; an expired QR code is refreshed. No account export or previously authorized session is supplied to an operator.
- `REL-02` (US-11): The provisioning component uses the authorized user account through MTProto, separate from the serverless message-routing path, to create a private forum supergroup for the product. That user account becomes the Telegram group owner.
- `REL-03` (US-11): If the product already has a support supergroup associated with it, provisioning does not create or replace the group.
- `REL-04` (US-11): Before asking Telegram to create a group, the backend stores a per-product provisioning operation with a unique marker and includes that marker in the new group description. On confirmed success, it records the group ID and product association before bot setup. The integration is not marked ready while the service bot or its required rights are missing.
- `REL-05` (US-11): Provisioning adds the common service bot to the created group, grants the rights required to receive manager messages, manage topics, and handle MVP manager join requests, and verifies those rights before reporting setup complete.
- `REL-06` (US-11): Shell displays a concrete failure reason when QR authorization, group creation, bot addition, or rights verification fails. Authorized-session storage, any additional verification, and a fallback when QR login cannot be used remain governed by Open Question 12.
- `REL-07` (US-11): If group creation has an unknown outcome, another group is not created automatically. Using the authorized account, the component lists its groups and checks their descriptions for the stored marker, reauthorizing by QR if necessary. A matching group is linked to the product and setup resumes. If the result remains unverified, Shell shows that review is required before a new creation attempt. The temporary marker is removed after successful setup.

## 7. Reliability, security, and data retention

Webhook processing is idempotent within the seven-day processed-update retention period. Topic recovery preserves the conversation when a topic is closed or deleted. Unknown results of topic or group creation are not retried automatically and require reconciliation. Outbound delivery attempts are stored before Telegram is called, unknown outcomes are not retried automatically, and delivery statuses are retained for 30 days. Administrative setup and readiness responses do not expose Telegram secrets or customer messages.

- `PRO-46` (US-03, Open Question 4): Conversation-to-topic associations are retained until the corresponding Telegram integration is deleted.
- `PRO-47` (Open Question 6): Development and demonstration use separate Telegram support bots, common service bots, support supergroups, webhook endpoints, and D1 databases. Local development uses the test resources; production resources are not prepared for the Prototype. Tokens and resource IDs are assigned during deployment.

## 8. Scope boundaries and open questions

- **Prototype:** Telegram Business, Shell setup, CSV onboarding, automatic invitations, and MTProto provisioning are outside scope; operators prepare groups and managers manually.
- **MVP:** Shell can connect an existing support supergroup, but does not create one.
- **Release:** MTProto creates the support supergroup. Manager onboarding still uses the MVP invitation-link flow; direct MTProto invitations are outside scope.
- **Across planned stages:** Telegram reply-to links are not preserved. Albums, edits, separate tickets, and internal notes in customer topics are outside the accepted scope.

The remaining decisions are tracked in [_docs/open-questions.md](_docs/open-questions.md): Business reply permissions (8), CSV columns and link distribution (9), Shell product identity and access (10), and user-session storage or QR fallback (12).

## 9. Traceability

This table maps User Stories and cross-cutting work to requirements and GitHub Issues. Prototype Issues have been reconciled against the current requirements; MVP and Release Issues have not been drafted.

| User Story | Specification requirements | GitHub Issues |
| --- | --- | --- |
| US-01 | PRO-01–PRO-07 | #2, #3, #4 |
| US-02 | PRO-08–PRO-12 | #3, #4 |
| US-03 | PRO-13–PRO-20, PRO-46, and PRO-48–PRO-49 | #2, #5, #6, #8, #10 |
| US-04 | PRO-21–PRO-27 | #6, #7, #8 |
| US-05 | PRO-28–PRO-37 | #5, #7, #8, #9, #10 |
| US-06 | PRO-38–PRO-45 | #3, #4, #6 |
| Cross-cutting Prototype environment | PRO-47 | #1, #11 |
| US-07 | MVP-01–MVP-06 | MVP issues not drafted |
| US-08 | MVP-07–MVP-10 | MVP issues not drafted |
| US-09 | MVP-11–MVP-13 | MVP issues not drafted |
| US-10 | MVP-14–MVP-18 | MVP issues not drafted |
| US-11 | REL-01–REL-07 | Release issues not drafted |
