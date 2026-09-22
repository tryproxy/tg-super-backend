# Decisions

Cross-task product and architecture decisions:

- The named stages are **Prototype**, **MVP**, and **Release**, in that order.
- Prototype is a backend demonstration of the complete support conversation through a regular Telegram bot. Telegram Business belongs to MVP.
- A product exists independently of this Telegram backend. The backend stores a reference to the product and its Telegram integration; it does not create or own the product. The authoritative Shell identifier remains an MVP question.
- Prototype supports multiple products. Each product has one active private forum supergroup and one dedicated Telegram support bot for this service; preserving another backend or webhook for that bot is not required.
- One platform service bot is the internal bridge and administrator in every product support group. It is distinct from the dedicated Telegram support bots.
- A conversation is identified by product ID, customer channel ID, and external customer chat ID. It has one active mapped topic in the product's group. A closed topic is reopened; a deleted topic is replaced and the mapping updated.
- A customer topic uses the customer's current Telegram display name when available and falls back to `Client <chat ID>`. A later display-name change updates the topic title; title changes do not change conversation identity.
- The group owner account, dedicated Telegram support bot or Business account, and manager accounts are separate roles; the Business account may differ from the group owner.
- Prototype groups and manager membership are set up manually. CSV onboarding, invite links, and Shell configuration/status belong to MVP.
- Prototype configuration is exposed through a protected admin API; Shell is not required to use it yet.
- An ordinary message sent by a participant in a mapped customer topic is immediately sent to the customer through the same Telegram support bot the customer contacted. Internal discussion belongs outside customer topics.
- Prototype supports text, photos up to 10 MB, and documents and voice messages up to 20 MB. The product support bot and service bot cannot reuse each other's `file_id`, so media is downloaded by one bot and uploaded by the other. Supported forwarded content is relayed without forwarding metadata or reply linkage. An unsupported first message still creates a topic; the topic shows the reason it was not relayed, and the sender receives a notice. Known delivery failures appear in the topic.
- Telegram webhook retries are deduplicated while processed update IDs are retained for 7 days; conversation mapping prevents duplicate active topics. Delivery statuses are kept for 30 days and mappings until integration deletion. If an external send has an unknown outcome, the mapped topic shows a warning and the backend does not retry automatically. A manager may explicitly send a new message, accepting that the customer may already have received the first one.
- Prototype has separate test and demo bot identities, webhook endpoints, forum groups, and D1 databases. Local development uses the test resources; production resources are deferred. Credentials and resource IDs are assigned at deployment.
- The Prototype prioritizes serverless operation and free service tiers: Cloudflare Workers Free with D1 Free, TypeScript, Hono, grammY, Zod, Wrangler, Vitest, Node.js and pnpm for local tooling.
- MVP adds Telegram Business, Shell integration, and CSV onboarding with invitations. A product may use its regular bot and Business channel at the same time, with separate conversation keys/topics. Business accounts use one shared platform connector bot, distinct from the internal service bot.
- Business connection is linked to a product through one-time pairing; an account already connected to another Business bot is refused in MVP. Manual replies sent by the Business account should appear in the topic as already sent, without redelivery. The account owner selects accessible chats in Telegram Business settings. Telegram enforces that selection; the backend does not fetch or store a separate recipient list and handles every business message delivered for the connection.
- MVP does not imply automatic group creation.
- MTProto group provisioning and direct manager invitations belong to Release and remain isolated from ordinary message routing.

Accepted assumptions and decisions are recorded with their questions in [open-questions.md](open-questions.md); unanswered MVP and later questions remain open. Any later change to a cross-task rule should update this file and the affected GitHub Issues or current stage documents.
