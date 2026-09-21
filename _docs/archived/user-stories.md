# User stories

The stage label indicates when a story is planned. Prototype is the first working demonstration; MVP follows it. Stories after MVP are not assigned to a third named stage.

## Prototype

### US-01 — Register product support

As an operator, I can register multiple products through a protected admin API and associate each with one product-owned bot and one existing forum supergroup.

Acceptance: the configuration is validated before activation; a group cannot be active for two products; credentials are not exposed by read endpoints.

### US-02 — Prepare the support group

As an operator, I can manually add the common service bot and managers to a product group so the team can work there.

Acceptance: the service bot has the permissions required to read messages and create topics; missing permissions produce an actionable setup status.

### US-03 — Start and continue a conversation

As a customer, I can message a product bot and have my messages appear in one active mapped topic for that product/channel/chat.

Acceptance: the first message, including an unsupported format, creates at most one active topic; later messages reuse it unless it was deleted, in which case a replacement is mapped; another product cannot receive the conversation; repeated webhook updates within the 7-day deduplication window do not duplicate delivery.

### US-04 — Send ordinary media

As a customer or manager, I can send text, photos, documents, and voice messages through the bridge when the cloud Bot API supports their transfer.

Acceptance: captions are preserved where applicable; supported forwarded content is relayed without forwarding metadata or reply linkage; unsupported or oversized content is explained in the topic and the sender is notified rather than shown a success claim.

### US-05 — Answer from a topic

As a manager, I can send a message in a mapped customer topic and the customer receives it from the product bot immediately.

Acceptance: replies from another group or unmapped topic are not delivered; the service bot's own messages are not echoed; a confirmed delivery failure appears in the same topic, while an unknown outcome is exposed for manual reconciliation without automatic resend.

### US-06 — Observe setup and delivery through the admin API

As an operator, I can see whether a product's bot, group permissions, webhook, and recent delivery are usable without a Shell interface.

Acceptance: status identifies the failing part without exposing credentials or customer content.

## MVP

### US-07 — Connect a Telegram Business account

As a product owner, I can connect a Business account through the shared platform connector bot so the support group handles its customer conversations and replies use the Business identity.

Acceptance: one-time pairing associates the connection with the product; the connection and granted rights are verified; an account connected to another Business bot is refused; the ordinary bot may remain active for the same product with separate customer topics; manual Business replies appear as already sent and are not redelivered. Business chat scope still needs a decision.

### US-08 — Configure support in Shell

As a product owner, I can set up a product channel and group in Runtime MF Shell and see actionable integration status.

Acceptance: the Shell calls backend APIs and never receives persistent Telegram credentials after registration.

### US-09 — Import managers from CSV

As a product owner, I can import expected managers for a product from CSV.

Acceptance: invalid rows are reported; email is not treated as Telegram identity; the CSV contract is defined before implementation.

### US-10 — Invite and identify managers

As a product owner, I can issue controlled invite links or join requests and associate the joined Telegram user with the expected manager record.

Acceptance: join status and role are visible; a manager's Telegram user ID is captured after joining; role privileges are limited to what is needed.

## After MVP

### US-11 — Provision a support group

As a product owner, I can authorize a Telegram user session to create and configure a forum group automatically.

### US-12 — Attempt direct manager invitations

As a product owner, I can request direct Telegram invitations where permitted, with a link-based fallback when Telegram prevents a direct invite.

Further capabilities such as separate tickets, message edits, albums, and internal notes inside customer topics remain uncommitted.
