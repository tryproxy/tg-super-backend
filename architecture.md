# Architecture

## Prototype shape

The Prototype uses a serverless webhook path. One Cloudflare Worker hosts the Hono admin API and the Telegram webhook handlers. D1 stores products, customer-channel references, support groups, conversations, topic mappings, processed update IDs, and delivery status. The Worker uses grammY to call the Telegram Bot API.

~~~text
Customer -> product bot -> Telegram webhook -> Worker -> D1
                                              -> service bot -> product forum topic
Manager -> forum topic -> service bot webhook -> Worker -> product bot -> Customer
~~~

Node.js and pnpm are local build and test tools. The deployed Worker does not require a continuously running Node.js server.

## Telegram identities

- A product-owned bot is the customer-facing identity for one product in Prototype.
- One platform service bot handles all internal forum groups and topic operations.
- A manually created owner account governs each group; the backend does not log in as that user.
- Managers join the appropriate group manually.

The routing key is (product ID, customer channel ID, external customer chat ID). The reverse lookup uses both group ID and topic ID. Database uniqueness and update deduplication protect these mappings.

Because the product bot and service bot are distinct, a file_id from one bot cannot be reused by the other. Supported media therefore needs a download and upload step; the cloud Bot API download ceiling is 20 MB. [Telegram Bot API](https://core.telegram.org/bots/api)

## Setup and secrets

A protected admin API registers products, dedicated customer bot credentials, and existing forum groups. The operator verifies bot access and service-bot group permissions before activation. A bot with another active backend or webhook is outside Prototype setup. There is no Shell UI in Prototype.

The admin API uses a service token held in a Worker secret. Product bot tokens are stored encrypted with a key held in a Worker secret; D1 must not contain plaintext tokens. Webhook endpoints validate Telegram's secret token and the receiving bot identity. Read APIs and logs must not expose credentials or customer message contents.

## Reliability

Telegram may resend webhook updates, so retain processed update IDs for 7 days and use unique conversation/topic keys. Persist an in-progress delivery attempt before calling Telegram. D1 and Telegram cannot commit atomically: if an attempt remains unfinished after an interruption, expose its outcome as unknown through the admin API, do not retry automatically, and require the operator to reconcile it manually. This avoids claiming exactly-once external delivery. A manager reply is accepted only from a mapped product group and topic. Service-bot messages and delivery-error notices must not be bridged back to customers. Confirmed outbound failure produces a notice in the same topic and a stored failure state; an unknown outcome is not presented as a confirmed failure.

The Prototype runs directly from webhook to D1 and Telegram Bot API. A queue or per-conversation serialization component is introduced only if implementation demonstrates a concrete retry or concurrent topic-creation problem.

## Operational data retention

Remove processed update IDs after 7 days and delivery statuses after 30 days. Keep the customer-to-topic mapping until its integration is deleted. Cleanup runs as scheduled Worker work; it does not require a continuously running server.

## Environment isolation

Prototype uses separate test and demo customer bots, service bots, webhook endpoints, forum groups, and D1 databases. Local development uses the test resources. Production resources are deferred until after Prototype; concrete credentials and resource IDs are assigned at deployment.

## Free-tier cost and limits

Checked against Cloudflare's published limits on 2026-09-20. Free is an account plan with usage ceilings, not unlimited production capacity.

| Service | Free allowance relevant here | If exceeded |
| --- | --- | --- |
| Workers Free | 100,000 requests/day; 10 ms CPU time per invocation | Requests can fail until the daily reset or require a paid plan. |
| D1 Free | 5 million rows read/day; 100,000 rows written/day; 5 GB total storage | Queries fail after daily row limits; storage must be freed or the account upgraded. |
| D1 Free database size | 500 MB per database; up to 10 databases/account | Data or database count must stay within the Free plan. |

Cloudflare's [Workers pricing](https://developers.cloudflare.com/workers/platform/pricing/) and [Workers limits](https://developers.cloudflare.com/workers/platform/limits/) document the compute limits. [D1 pricing](https://developers.cloudflare.com/d1/platform/pricing/) and [D1 limits](https://developers.cloudflare.com/d1/platform/limits/) document the database limits. The 10 ms CPU ceiling needs a real webhook smoke test with Hono, grammY, and D1; external network waiting does not make that CPU allowance larger.

Workers Paid currently starts at USD 5/month and provides higher usage allowances, but no paid Cloudflare service is required to start this Prototype. No alternative database is needed while D1 Free satisfies the expected prototype workload. This cost assessment should be rechecked before deployment because platform pricing can change.

## MVP and later boundaries

MVP adds a Telegram Business adapter, Shell configuration/status, and CSV manager onboarding with invitations. A product may use both customer channels at once. One shared platform Business connector bot handles Business connections; it is separate from the internal service bot. Both channels normalize into the same conversation model, while channel ID keeps their customer topics separate. A one-time pairing flow links a Business connection to its product. An account already connected to another Business bot is refused in MVP; chat scope and required rights still need decisions before implementation. Manual Business replies should be shown as already sent rather than sent back to the customer again.

Automated creation of user-owned groups and direct manager invitation require a separately authorized MTProto component. They are ideas for after MVP, not part of the two named stages.
