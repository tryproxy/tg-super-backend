# User Stories

Status: Draft. Stories marked **MVP** describe the current proposed first release; the remaining stories preserve discussed follow-up options.

## Product setup

### US-01 — Register Telegram support for a product **MVP**

As a product owner, I want to register a Telegram customer channel and support group for my product so that customer conversations are routed to the correct team.

Acceptance notes:

- The configuration belongs to exactly one product.
- The backend validates access to the bot and group before activation.
- The Shell can show whether the integration is ready or needs attention.

### US-02 — Use a product-owned bot

As a product owner, I want to connect a bot that I control so that I retain its public username and ownership.

Acceptance notes:

- The bot token is stored as a secret and is never returned to the browser after registration.
- A token or webhook conflict produces an actionable setup status.
- Disconnecting the product does not silently destroy or transfer the bot.

### US-03 — Use a platform-managed bot

As a product owner, I want the platform to provide and configure a bot so that I can start support without operating BotFather settings or webhooks.

Acceptance notes:

- Ownership and offboarding behavior are visible before activation.
- Each incoming update can be resolved to the correct product.

### US-04 — Connect a Telegram Business account

As a product owner, I want to connect my Telegram Business account so that customers communicate with the account's identity while the same support workflow handles the messages.

Acceptance notes:

- The owner grants access through Telegram and does not provide account credentials or a user session.
- The backend stores the business connection and granted rights.
- The Shell reports disconnected, insufficient-rights, and connected states.
- Business messages use the same conversation/topic workflow as regular bot messages.

## Support group provisioning

### US-05 — Register an existing forum supergroup **MVP**

As a product owner, I want to attach an existing forum supergroup so that managers can use it as the product's support inbox.

Acceptance notes:

- The group is forum-enabled.
- The support bot is an administrator with the rights needed to read manager messages and manage topics.
- The group is not already active for an incompatible product configuration.

### US-06 — Automatically create a support group

As a product owner, I want an authorized Telegram account to create and configure the product's support supergroup so that setup requires minimal manual work.

Acceptance notes:

- The owner explicitly authorizes the user-account session used for provisioning.
- The created group has forum topics enabled.
- The support bot is added with the required administrator rights.
- Provisioning failures are resumable and visible without exposing session secrets.

## Manager onboarding and access

### US-07 — Import product managers from CSV **MVP**

As a product owner, I want to import a list of managers so that the expected support team is associated with the product.

Acceptance notes:

- A row can contain a name, email, optional Telegram username or phone, and requested role.
- Email is not treated as proof of Telegram identity.
- Invalid rows are reported without discarding valid rows.

### US-08 — Invite a manager through a personal link **MVP**

As a product owner, I want each imported manager to receive a controlled invitation so that the correct Telegram user joins the support group.

Acceptance notes:

- The invitation can expire or have a usage limit.
- Joining binds the manager record to the Telegram user ID.
- Reusing or revoking an invitation has a visible status.

### US-09 — Apply manager privileges **MVP**

As a product owner, I want managers to receive the minimum permissions for their requested role so that they can answer customers without unnecessary group control.

Acceptance notes:

- A normal manager can read and write in customer topics.
- Elevated rights are granted only when the configured role requires them.
- Removing a manager prevents future support actions and updates the integration status.

### US-10 — Attempt direct manager invitation

As a product owner, I want the provisioning account to attempt direct Telegram invitations so that eligible managers can join without using a link.

Acceptance notes:

- Privacy or Telegram-limit failures fall back to a personal invite link.
- A failed direct invitation is not reported as successful onboarding.

## Customer conversations

### US-11 — Start a customer conversation **MVP**

As a customer, I want my first message to reach the correct product team so that I can request support through Telegram.

Acceptance notes:

- The product is derived from the receiving customer channel.
- The first message creates at most one active customer topic.
- Duplicate webhook delivery does not duplicate the topic or message.

### US-12 — Continue in the same topic **MVP**

As a manager, I want subsequent messages from the same product/customer conversation to appear in the same topic so that the full context stays together.

Acceptance notes:

- The mapping includes product, customer channel, and external customer chat.
- The same Telegram user contacting another product does not reuse the wrong topic.

### US-13 — Answer a customer from Telegram **MVP**

As a manager, I want to answer from the customer's topic so that I can work without a separate support application.

Acceptance notes:

- Only messages from the configured product group and a mapped customer topic can be delivered externally.
- The customer receives the answer through the product's configured customer channel.
- Failed delivery is visible to managers or operators.

### US-14 — Receive customer media

As a manager, I want supported customer media to appear in the topic so that I can handle requests containing files, photos, voice messages, or other Telegram content.

### US-15 — Preserve reply context and edits

As a customer or manager, I want replies and supported edits to retain their context across the bridge so that the conversation remains understandable.

## Operations

### US-16 — Observe integration health **MVP**

As a product owner, I want the Shell to show bot/business connection, group, permission, webhook, and manager-onboarding status so that setup failures are actionable.

### US-17 — Recover a deleted or closed topic

As a manager, I want the system to reopen or recreate a missing conversation topic so that support can continue without manual database repair.

### US-18 — Keep internal notes private

As a manager, I want an explicit way to write an internal note so that team discussion cannot be accidentally delivered to the customer.
