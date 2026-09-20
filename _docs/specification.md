# Prototype specification

Status: Prototype scope; implementation has not started. Accepted Prototype assumptions and unresolved later questions are tracked in [open-questions.md](open-questions.md).

This document defines observable behavior for the Prototype. [plan.md](../plan.md) gives the product narrative; [decisions.md](decisions.md) records cross-task choices.

## Setup and access

- P-01: An operator can register more than one product through a protected admin API.
- P-02: For each product, the operator can register one dedicated product-owned bot token and one existing private forum supergroup. Prototype setup does not preserve another backend or webhook for that bot.
- P-03: Activation verifies that the customer bot is usable and the common service bot can read group messages and create forum topics. Invalid setup produces an actionable status.
- P-04: The same support group cannot be active for two products.
- P-05: Managers are added to the Telegram group manually. Their group membership, rather than an imported email address, establishes access in the Prototype.
- P-06: Secrets are not returned by read endpoints or written to logs.

## Customer messages

- P-07: A private message to a configured product bot resolves to that product and its customer channel.
- P-08: The first message from a customer creates one topic in the product's forum group and persists the conversation-to-topic mapping, including when its format is unsupported.
- P-09: Later messages from the same customer to the same channel reuse the active mapped topic. A closed topic is reopened; a deleted topic is replaced and the mapping updated. Another product or channel gets a separate conversation. The topic title uses the customer's Telegram display name, or `Client <chat ID>` when no name is available. If the display name later changes, the topic title is updated without creating a new conversation.
- P-10: Text, photos, documents, and voice messages are relayed with their captions where applicable. Supported content from forwarded messages is treated as ordinary content; forwarding metadata and reply linkage are not preserved. For media that requires download and upload between the product bot and service bot, the cloud Bot API 20 MB download ceiling applies.
- P-11: Unsupported or oversized content produces a topic notice explaining what was not relayed, including for a first message. A customer sender is notified through the product bot; a manager sender sees the notice in the topic. The system does not claim successful delivery.
- P-12: Repeated webhook updates are deduplicated within the 7-day processed-ID retention window and do not create additional active topics or duplicate messages. An outbound attempt with an unknown result is not resent automatically.

## Manager replies

- P-13: An ordinary message sent in a mapped customer topic by a group participant is treated as an immediate reply to the customer.
- P-14: The reply is delivered through the same product bot the customer contacted. The customer sees the bot identity.
- P-15: Messages from another group, an unmapped topic, or the service bot's own output are not sent to a customer.
- P-16: A confirmed delivery failure produces a clear notice in the same topic without creating a reply loop. An interrupted attempt with an unknown result is visible to the operator through the admin API and requires manual reconciliation; it is not automatically retried.
- P-17: Internal discussion takes place outside mapped customer topics.

## Operational data

- P-18: Persist an in-progress delivery attempt before calling Telegram. Expose attempts with unknown outcomes through the protected admin API for manual reconciliation.
- P-19: Retain processed update IDs for 7 days and delivery statuses for 30 days. Retain conversation-to-topic mappings until the integration is deleted.

## Prototype completion check

- Configure two products and two groups manually.
- Send first and subsequent messages to both bots; verify correct topics, media transfer, and isolation.
- Send manager replies from the mapped topics; verify each customer receives the reply from the correct bot.
- Repeat inbound updates within the retention window and simulate confirmed failures and interrupted outbound attempts; verify deduplication, visible failure, and an operator-visible unknown state without automatic resend.
- Send an unsupported first message and verify that it creates a topic, explains the rejection there, and notifies the sender.
- Close and delete mapped topics; verify reopening and replacement with an updated mapping.
- Verify an invalid bot token or missing group permissions prevents activation.
- Verify the flow runs through a webhook-based Cloudflare Worker and D1 without a continuously running backend process. Use separate test and demo Telegram resources; local development uses test resources.

## Scope boundary

Telegram Business, Shell UI, CSV import, generated manager invitations, automated group creation, albums, message edits, and internal-note commands are outside Prototype. Business, Shell and CSV onboarding are planned for MVP. Other items remain open for later prioritization.
