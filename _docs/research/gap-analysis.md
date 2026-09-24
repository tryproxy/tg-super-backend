# Gap analysis: Telegram support backend and web chat backend

**Status:** research and recommendations, not accepted product requirements. Reviewed on 2026-09-24. This report compares documentation and the current Prototype issue descriptions; it does not verify the behavior of `web-chat-backend` by running its code. Accepted behavior remains in [SPEC.md](../../SPEC.md), with reasons in [Decisions](../decisions.md) and unresolved choices in [Open Questions](../open-questions.md).

## Scope of the comparison

- `tg-super-backend`: all 22 Markdown files present during the review, including archived files as historical context. Its active material is `SPEC.md`, `README.md`, `AGENTS.md`, and the active `_docs/` files. Prototype [Issues #1–#11](https://github.com/tryproxy/tg-super-backend/issues?q=is%3Aissue+label%3Aprototype) were checked to distinguish missing work from work already planned.
- `web-chat-backend`: all 10 Markdown files in the local `dev` checkout, including its README, Cloudflare handoff, architecture, operations, troubleshooting, credentials audit, incidents, MCP read API, and AI research. The source repository is [AA-asomarket/web-chat-backend](https://github.com/AA-asomarket/web-chat-backend/tree/dev).
- Findings about that repository describe its **documentation**. In particular, [AI support research](https://github.com/AA-asomarket/web-chat-backend/blob/dev/docs/AI_SUPPORT_RESEARCH.md) includes proposals and should not be read as proof that every proposed feature is deployed.

## What is already covered here

The current Prototype already specifies one dedicated customer-facing bot and one private forum supergroup per product, a shared service bot, explicit customer-to-topic identity, a protected admin API, manually prepared groups and managers, supported media, topic recovery, webhook update deduplication, an attempt record before manager-to-customer sends, seven-day processed-update retention, and 30-day delivery-attempt retention. Test and demo resources are separate. The selected stack is Workers, D1, Hono, grammY, Zod, Wrangler, Vitest, TypeScript, Node.js, and pnpm.

Issues already plan `/health` (#1), topic-creation recovery (#6), manager delivery recovery (#9), scheduled cleanup (#10), and a two-product demonstration (#11). The gaps below refine those contracts; they do not imply that these capabilities are absent from the backlog. Telegram Business and Shell configuration belong to MVP; account-based MTProto group creation belongs to Release.

## Prototype gaps and recommendations

| ID | Priority | Observation in current plan | Recommended adjustment | Affected existing work |
| --- | --- | --- | --- | --- |
| G1: interrupted webhook | Resolve before message processing | [Issue #5](https://github.com/tryproxy/tg-super-backend/issues/5) stores the first update claim before downstream work, then acknowledges a repeated claim without a second downstream call. It does not say how interrupted work completes. | Define received, processing, completed, and needs-review outcomes. A repeated webhook should inspect the recorded outcome; it should resume only work that can be repeated safely. Define the HTTP response and recovery behavior for each outcome. | `SPEC.md` `PRO-19`, architecture; Issues #2 and #5, with tests in #6–#9. |
| G2: customer-to-topic posting | Resolve before inbound routing | The spec explains unknown topic creation and manager-to-customer delivery. After a topic exists, failure or an unknown result while posting the customer's message into it has no equivalent contract. | Specify known failure, unknown result, customer notice, and operator recovery. Decide whether an inbound delivery attempt is stored before posting. Do not claim that the topic or manager received content when posting is unconfirmed. | `SPEC.md`, Decisions or Open Questions, architecture; Issues #2, #6, #8, and possibly #9. |
| G3: warning delivery failure | Clarify for Prototype | Issue #9 warns managers in the topic about unknown customer delivery; the common service bot may itself have lost group access. | Preserve the original attempt outcome, record whether posting the warning failed, and provide safe diagnostic information for an operator. The warning is a best effort action, not a guaranteed notification. | Architecture; Issue #9 and demonstration checks in #11. |
| G4: operational explanation | Complete with the Prototype | [Operations](https://github.com/AA-asomarket/web-chat-backend/blob/dev/docs/OPERATIONS.md) and [Troubleshooting](https://github.com/AA-asomarket/web-chat-backend/blob/dev/docs/TROUBLESHOOTING.md) distinguish deployment, webhook, Telegram, storage, and authorization failures. Our README has no corresponding practical guide yet. | Once commands exist, document test/demo deployment targets, migration order, required secret *names*, smoke checks, logs, and recovery from a bad deployment. Provide a short symptom-to-check map for missing customer messages, missing manager replies, and unavailable topic notices. | README, architecture if needed; Issue #11. |
| G5: free-tier capacity evidence | Verify in the demonstration | [Tech Stack](../tech-stack.md) selects free tiers, but the planned demonstration does not establish whether media transfer and scheduled D1 scans fit the intended resource limits. | Exercise a supported large media transfer and inspect Worker CPU/memory and D1 reads/writes. Define the visible behavior when a provider limit is hit. Reassess a limit or architecture only from observed evidence. | Architecture and Issue #11; leave the selected stack as is. |
| G6: data-storage boundary | Decide with G1–G2 | The reference stores message history in D1 for its inbox and AI features. Our architecture lists routing metadata and delivery attempts, but does not explicitly state whether customer or manager message bodies are stored. | State the Prototype content-retention policy. A metadata-only attempt record cannot replay an original message once its webhook payload is unavailable. Choosing automatic recovery would require a temporary payload or another durable source, with retention and access rules. | `SPEC.md` reliability section, architecture, Decisions or Open Questions; Issues #2, #5, #6, #8 as needed. |

### Failure paths that need distinct outcomes

1. **Claim saved, no Telegram call started, Worker stops.** A later webhook should not be skipped merely because its update ID exists. Safe continuation requires enough state or the repeated webhook payload to finish the work.
2. **Telegram call may have succeeded, result not recorded.** Blindly sending again can duplicate a customer message, topic, or manager reply. Preserve an unknown state and reconcile the particular side effect. D1 and Telegram do not share one transaction.
3. **Telegram confirmed failure.** Record the failure and notify the affected side when that notification can be sent. A failed notice must itself be diagnosable.
4. **Webhook payload no longer available.** Without stored content, the backend cannot recreate a missing customer message. The product contract must say whether the customer is asked to resend or whether content is temporarily stored for recovery.

**Recommended minimal Prototype assumption, not yet accepted:** store routing state and attempt metadata but no full conversation transcript in D1; for an inbound post with an unknown outcome, avoid automatic replay and tell the customer that transfer is unconfirmed and a resend may be needed. This gives a narrower reliability guarantee than automatic recovery. If the desired guarantee is eventual delivery of the original message without customer action, decide on temporary content storage before updating the issues. A delivery-attempt record alone is insufficient.

The documented rule for manager-to-customer unknown outcomes already prohibits automatic resend. Extend it to inbound posting only after confirming the desired customer experience and operator action. The existing protected operational API in Issue #6 covers **uncertain topic creation**; extending it to message delivery would be a new scope decision. Issue #9 explicitly excludes an admin delivery-history API, so the report does not assume one.

## MVP gaps to resolve before grooming its issues

| ID | Observation | Suggested question or rule | Where it belongs |
| --- | --- | --- | --- |
| G7: integration lifecycle | Registration and initial readiness are well defined; disabling support, rotating a bot token, disconnecting Business, and replacing a group are not. | What happens to webhooks, pending attempts, existing topic mappings, and customer-visible status for each operation? Which changes require a new mapping? | Add to Open Questions for MVP; then define behavior in `SPEC.md` and reason in Decisions. |
| G8: product access in Shell | Prototype uses one operator credential. The reference's admin API relies on an upstream service to validate an administrator; that is its own project-specific contract. | Which Shell product or remote ID is authoritative, and who may read or change each integration? Check product scope on every relevant API action. | Extend existing Open Question 10, then `MVP-07`–`MVP-10`. |
| G9: work visibility | The reference inbox exposes whether a thread waits for a reply. Our managers currently work directly in Telegram topics. | Decide whether managers need a separate “waiting for support” view or counts. If yes, define which message and delivery outcomes change that state. | Optional MVP product discovery; add a User Story only if chosen. |

## Possible later extensions, with their costs

| Idea seen in the reference | Possible value here | Additional decision required |
| --- | --- | --- |
| Aggregate workload metrics | Count unanswered conversations and response time by product without necessarily storing transcripts. | Define what counts as a reply, including failed and unknown sends. |
| Searchable support history | Search or review conversations outside Telegram. | Store message contents, define retention and access by product, and handle media and deletion. |
| AI drafts or summaries | Assist managers with repeated questions. | Approve the product goal, message-data access, cost, manager review, and separation between internal and customer-visible information. The reference's AI research is a proposal, not a Prototype dependency. |
| Read-only cross-system support API | Feed a later authorized support workspace. | Define authentication and product-scoped authorization first. The reference's [MCP read API](https://github.com/AA-asomarket/web-chat-backend/blob/dev/docs/MCP_READ_API.md) is an example of scoped access, not a required component. |

These extensions do not justify adding a support inbox, Durable Object stream, Queue, R2, or Workers AI to the present Prototype. The reference has a web widget and other workflows that our current product does not have. Its Cloudflare handoff describes a Durable Object for the live stream even in its minimal configuration; our Telegram topic workflow has no live web stream requirement.

## Cautions when borrowing from the reference

- Its [architecture binding list](https://github.com/AA-asomarket/web-chat-backend/blob/dev/docs/ARCHITECTURE.md) does not list the Durable Object described as required in its [handoff guide](https://github.com/AA-asomarket/web-chat-backend/blob/dev/README_HANDOFF_CLOUDFLARE.md). Check code and deploy configuration before treating either as an implementation fact.
- Its [credentials audit](https://github.com/AA-asomarket/web-chat-backend/blob/dev/docs/CREDENTIALS_AUDIT.md) and troubleshooting guide name different admin-auth endpoints or examples. Their authentication adapter is specific to their main backend; it does not establish our future Shell authorization contract.
- Its documentation flags exposed debug routes and secrets configured as plain vars as operational risks. Borrow the diagnostic questions, not the exact debug routes or credentials layout.
- Cloudflare plan limits and prices can change. Check [Workers limits](https://developers.cloudflare.com/workers/platform/limits/) and [D1 pricing](https://developers.cloudflare.com/d1/platform/pricing/) before deploying or changing the approved stack. Telegram can retry unsuccessful webhooks as described in the [Bot API](https://core.telegram.org/bots/api#setwebhook).

## Suggested order of follow-up

1. Decide the inbound content-retention and unknown-delivery policy (G2 and G6). Keep the answer as a new Open Question until accepted; then record the decision and observable behavior in `SPEC.md`.
2. Specify interrupted-update recovery (G1) and fallback diagnostics (G3) against that policy. Add tests for interruption before a Telegram call, after a possible Telegram success, and after a failed warning.
3. Reconcile affected Prototype Issues #2, #5, #6, #8, #9, and #11. Update [User Stories](../user-stories.md) only if the customer- or manager-visible promise changes; keep implementation details in `SPEC.md` and Issues. Remove the duplicated “The file passes under `pnpm test`” sentence from Issue #9 during grooming.
4. Complete the small operational guide and capacity smoke checks with Issue #11, using the actual commands and deployment resources created during implementation.
5. Resolve MVP lifecycle and Shell access (G7–G8) before drafting MVP Issues. Evaluate workload metrics, history, and AI only when there is a product need.

**No current decision is changed by this research file.** Its assumptions and proposed issue edits require reconciliation with active documents before implementation.

## How to record a new gap

During grooming, first check the active SPEC, Decisions, Open Questions, and current Issue. If they already define the behavior and only the Issue is outdated, correct that Issue; this is not a new gap. For a newly verified gap, use the next unused `G` number and record: the observed mismatch or missing case, evidence, affected stage and Issue, the proposed choice, and whether the answer is open or already accepted. Place it before or inside the affected open Issue in the [Gap Implementation Roadmap](gap-implementation-roadmap.md), or after Prototype when appropriate. If that Issue is already closed, propose a follow-up Issue; reopening is appropriate only for an unmet accepted criterion.

A gap blocks handoff only when the current Issue cannot have clear, consistent criteria without resolving it. Accepted behavior belongs in SPEC and Decisions, and in the affected Issue; this research file does not become the implementation contract. GitHub Issue state records whether implementation and QA are complete.
