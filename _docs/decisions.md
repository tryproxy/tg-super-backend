# Decisions

Cross-task product and architecture decisions:

- The named stages are **Prototype** and **MVP**. Work after MVP has no named stage yet.
- Prototype is a backend demonstration of the complete support conversation through a regular Telegram bot. Telegram Business belongs to MVP.
- Prototype supports multiple products. Each product has one active private forum supergroup and one product-owned customer-facing bot dedicated to this service; integration with an existing bot backend or webhook is not required.
- One platform service bot is the internal bridge and administrator in every product support group. It is distinct from customer-facing product bots.
- A conversation is identified by product ID, customer channel ID, and external customer chat ID. It has one active mapped topic in the product's group. A closed topic is reopened; a deleted topic is replaced and the mapping updated.
- A customer topic uses the customer's current Telegram display name when available and falls back to `Client <chat ID>`. A later display-name change updates the topic title; title changes do not change conversation identity.
- The group owner account, customer-facing bot or Business account, and manager accounts are separate roles; the Business account may differ from the group owner.
- Prototype groups and manager membership are set up manually. CSV onboarding, invite links, and Shell configuration/status belong to MVP.
- Prototype configuration is exposed through a protected admin API; Shell is not required to use it yet.
- An ordinary message sent by a participant in a mapped customer topic is immediately sent to the customer through the product bot. Internal discussion belongs outside customer topics.
- Prototype supports text, photos, documents, and voice messages up to the cloud Bot API download limit when forwarding requires a download. Supported forwarded content is relayed without forwarding metadata or reply linkage. An unsupported first message still creates a topic; the topic shows the reason it was not relayed, and the sender receives a notice. Known delivery failures appear in the topic.
- Telegram webhook retries are deduplicated while processed update IDs are retained for 7 days; conversation mapping prevents duplicate active topics. Delivery statuses are kept for 30 days and mappings until integration deletion. An uncertain external send is not retried automatically and requires operator reconciliation.
- Prototype has separate test and demo bot identities, webhook endpoints, forum groups, and D1 databases. Local development uses the test resources; production resources are deferred. Credentials and resource IDs are assigned at deployment.
- The Prototype prioritizes serverless operation and free service tiers: Cloudflare Workers Free with D1 Free, TypeScript, Hono, grammY, Zod, Wrangler, Vitest, Node.js and pnpm for local tooling.
- MVP adds Telegram Business, Shell integration, and CSV onboarding with invitations. A product may use its regular bot and Business channel at the same time, with separate conversation keys/topics. Business accounts use one shared platform connector bot, distinct from the internal service bot.
- Business connection is linked to a product through one-time pairing; an account already connected to another Business bot is refused in MVP. Manual replies sent by the Business account should appear in the topic as already sent, without redelivery. The set of customer chats covered by the connection remains open.
- MVP does not imply automatic group creation.
- MTProto group provisioning and direct manager invitations are considered after MVP and remain isolated from ordinary message routing.

Accepted Prototype assumptions are recorded with their questions in [open-questions.md](open-questions.md); MVP and later questions remain open. Any later change to a cross-task rule should update this file and the affected specification.
