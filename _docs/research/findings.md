# Research Findings and implementation order

**Status:** active research registry and schedule. Reviewed on 2026-09-24.

This document consolidates the earlier separate research report and implementation schedule. It compares the current Telegram support plan with useful ideas found in `web-chat-backend`, then places each late discovery against the existing backlog. The comparison concerns that repository's documentation; proposals in its AI research are not proof of deployed behavior.

## Authority

Findings provide evidence, recommendations, and ordering. Accepted behavior remains in [SPEC.md](../../SPEC.md), explanations in [Decisions](../decisions.md), unresolved product choices in [Open Questions](../open-questions.md), and implementation contracts in GitHub Issues. Follow the lifecycle in [Research workflow](README.md).

No Finding in this file changes a product decision by itself.

## Existing coverage

The Prototype already specifies one dedicated customer-facing bot and one private forum supergroup per product, a shared service bot, explicit customer-to-topic identity, a protected admin API, manually prepared groups and managers, supported media, topic recovery, webhook update deduplication, an attempt record before manager-to-customer sends, seven-day processed-update retention, and 30-day delivery-attempt retention. Test and demo resources are separate.

Issues already plan `/health` (#1), topic-creation recovery (#6), manager delivery recovery (#9), scheduled cleanup (#10), and a two-product demonstration (#11). Findings refine those contracts; they do not mean the capabilities are absent from the backlog. Telegram Business and Shell configuration belong to MVP. Account-based MTProto group creation belongs to Release.

## Registry

| ID | Status | Stage and placement | Tracking | Decision state |
| --- | --- | --- | --- | --- |
| G1: interrupted webhook | `accepted` | Before #5; demonstrate inside #11 | [#12](https://github.com/tryproxy/tg-super-backend/issues/12) | Recovery rule remains open until #5 grooming. |
| G2: customer-to-topic posting | `accepted` | Before #6; inside #8; demonstrate inside #11 | [#13](https://github.com/tryproxy/tg-super-backend/issues/13) | Failure and unknown-result behavior remains open. |
| G3: warning delivery failure | `accepted` | Inside #7 and #9; demonstrate in #11 | [#14](https://github.com/tryproxy/tg-super-backend/issues/14) | Fallback must be accepted during grooming. |
| G4: operational explanation | `accepted` | Inside #11 | [#15](https://github.com/tryproxy/tg-super-backend/issues/15) | Write from implemented commands and resources. |
| G5: Free-tier capacity evidence | `accepted` | Inside #11 | [#16](https://github.com/tryproxy/tg-super-backend/issues/16) | Evidence pending; stack remains unchanged until measured. |
| G6: data-storage boundary | `accepted` | Before #6, together with G2; inside #8 | [#17](https://github.com/tryproxy/tg-super-backend/issues/17) | Message-content retention remains open. |
| G7: integration lifecycle | `proposed` | Before MVP integration-management Issues | — | Resolve before grooming that scope. |
| G8: product access in Shell | `proposed` | Before MVP Shell API Issues | — | Extend Open Question 10 before grooming. |
| G9: work visibility | `proposed` | Product discovery after Prototype | — | Optional; create a User Story only if chosen. |

`accepted` means the Finding must be handled at the recorded placement. It does not mean its recommended product answer has already been accepted.

## Prototype queue

1. **[#1 — bootstrap the Worker](https://github.com/tryproxy/tg-super-backend/issues/1):** implement and close. No Finding precedes it.
2. **[#2 — D1 schema](https://github.com/tryproxy/tg-super-backend/issues/2):** implement the current task. Fields later required by G1, G2, or G6 may be added by a migration in #5 or #6; they do not delay #2.
3. **[#3 — register integrations](https://github.com/tryproxy/tg-super-backend/issues/3):** implement and close. G7 and G8 belong to MVP.
4. **[#4 — validate and activate](https://github.com/tryproxy/tg-super-backend/issues/4):** implement and close. No Finding precedes it.
5. **G1 → #5:** resolve interrupted-update behavior during grooming, reconcile `PRO-19`, architecture, and #5, then implement it in #5.
6. **G2 + G6 → #6:** decide inbound delivery failure behavior and content retention before #6 handoff. Implement the accepted text path and any required migration in #6.
7. **#7 with G3:** include failed manager-warning delivery in #7's criteria and diagnostics.
8. **#8 with G2 and G6:** apply the accepted inbound policy to files. No new product decision is expected unless media exposes a case not covered by the earlier answer.
9. **#9 with G3:** preserve the original unknown delivery result and diagnose a failed topic warning without a second customer send.
10. **#10:** apply retention rules to new states from #5–#9 and do not erase unresolved cases without diagnostics.
11. **#11 with G4 and G5:** write the operational guide from real commands, measure supported large-media and D1 work, and demonstrate the failure paths introduced by G1–G3.

Linear order: **#1 → #2 → #3 → #4 → G1 → #5 → G2/G6 → #6 → #7 with G3 → #8 → #9 with G3 → #10 → #11 with G4/G5**.

Do not pause #1–#4 to solve later Findings.

## Prototype Findings

### G1: interrupted webhook

- **Source and evidence:** #5 stores the first update claim before downstream work, then acknowledges a repeated claim without saying whether the first attempt completed. Telegram can retry unsuccessful webhooks.
- **Impact:** reliability and recovery.
- **Placement:** before #5; downstream Telegram side effects are verified in #6–#9; demonstrate accepted recovery inside #11.
- **Affected work:** `SPEC.md` `PRO-19`, architecture, #5, and relevant checks in #6–#9.
- **Risk if skipped:** a repeated update can be discarded as a duplicate even though processing stopped, or unsafe recovery can repeat an external action.
- **Recommendation:** distinguish received, processing, completed, and needs-review outcomes. A repeat inspects the stored outcome and resumes only work that is safe to repeat. Define the HTTP acknowledgement for each outcome.
- **Implementation home:** #5 owns webhook state; #6–#9 own their Telegram side effects.
- **Complete when:** accepted rules distinguish unfinished, completed, and unknown external-action states; active requirements and #5 agree; tests prove that completed repeats do not repeat work and interrupted updates are not silently lost.

### G2: customer-to-topic posting

- **Source and evidence:** current requirements cover uncertain topic creation and manager-to-customer delivery, but not a failure or unknown result when posting a customer's message into an existing topic.
- **Impact:** customer-visible behavior, delivery state, diagnostics, and possibly persistence.
- **Placement:** before #6 for text; reuse the accepted rule inside #8 for media; demonstrate accepted recovery inside #11.
- **Affected work:** SPEC, Decisions or Open Questions, architecture, #2, #6, #8, and possibly #9.
- **Risk if skipped:** the service can claim that support received content when delivery was never confirmed, or blindly repeat a Telegram action that may already have succeeded.
- **Recommendation:** define confirmed failure, unknown result, customer notice, operator recovery, and whether an inbound attempt is stored before posting.
- **Implementation home:** #6 owns text and routing; #8 applies the policy to supported media. Any new admin API is a separate scope decision.
- **Complete when:** active requirements and #6/#8 agree; tests cover accepted failure and unknown-result behavior; no unconfirmed post is reported as delivered.

### G3: warning delivery failure

- **Source and evidence:** #9 warns managers in the topic about unknown customer delivery, but the service bot may itself have lost group access. The same problem can occur for a warning after a confirmed send failure in #7.
- **Impact:** diagnostics and recovery.
- **Placement:** inside #7 and #9; demonstrate in #11.
- **Affected work:** architecture, #7, #9, and #11.
- **Risk if skipped:** the original attempt outcome can be overwritten or operators can be told that managers were warned when no warning arrived.
- **Recommendation:** preserve the original attempt outcome, record whether the warning failed, and expose safe diagnostics. Treat the warning as best effort.
- **Implementation home:** #7 and #9 own delivery paths; #11 owns the demonstration. Do not add a standalone warning service.
- **Complete when:** the active documents and #7/#9 describe the fallback, focused checks cover failed warning delivery, and no fallback sends the customer message a second time.

### G4: operational explanation

- **Source and evidence:** the reference repository separates deployment, webhook, Telegram, storage, authorization, and symptom-based troubleshooting. This repository does not yet have implemented commands or an equivalent practical guide.
- **Impact:** operations and recovery.
- **Placement:** inside #11 after real commands and resources exist.
- **Affected work:** README, architecture when necessary, and #11.
- **Risk if skipped:** a working Prototype cannot be deployed, checked, or recovered consistently by someone without the implementation session.
- **Recommendation:** document test/demo targets, migration order, required secret names without values, smoke checks, logs, bad-deployment recovery, and a short symptom-to-check map for missing customer messages, manager replies, and topic notices.
- **Implementation home:** #11.
- **Complete when:** #11 QA follows the guide against the implemented resources and discrepancies are corrected.

### G5: Free-tier capacity evidence

- **Source and evidence:** Tech Stack selects Workers Free and D1 Free, but the planned demonstration does not yet measure supported large-media transfer or scheduled D1 work.
- **Impact:** deployment constraints and observable limit handling.
- **Placement:** inside #11.
- **Affected work:** architecture and #11; Tech Stack changes only if evidence requires it.
- **Risk if skipped:** the Prototype can pass functional tests while exceeding the intended free runtime or failing without defined user-visible behavior.
- **Recommendation:** exercise a supported large media transfer, inspect Worker CPU and memory plus D1 reads and writes, and record behavior when a provider limit is hit.
- **Implementation home:** #11.
- **Complete when:** #11 records observed usage, accepted limit behavior, and QA evidence from the actual test/demo setup.

### G6: data-storage boundary

- **Source and evidence:** architecture lists routing metadata and delivery attempts but does not explicitly state whether customer or manager message bodies are stored. Metadata cannot recreate an original payload after Telegram stops retrying it.
- **Impact:** product privacy, recovery guarantees, persistence, and retention.
- **Placement:** before #6 together with G2; apply the accepted policy inside #8.
- **Affected work:** SPEC reliability rules, architecture, Decisions or Open Questions, #2, #5, #6, and #8 as required by the chosen design.
- **Risk if skipped:** requirements may promise replay that metadata-only storage cannot perform, leading to a later schema and privacy redesign.
- **Recommendation:** state which content, if any, is stored, its retention and access, and whether a missing original can be replayed or the customer must resend.
- **Implementation home:** #6/#8 own routing and media; any required migration belongs in the affected implementation Issue, not the Finding tracking Issue.
- **Complete when:** the policy is accepted, active documents and #6/#8 agree, and tests prove only the recovery guarantee the chosen storage permits.

## Failure outcomes that must stay distinct

1. **Claim saved, no Telegram call started, Worker stops.** A later webhook must not be skipped solely because its update ID exists. Safe continuation needs state plus the repeated payload or another durable source.
2. **Telegram call may have succeeded, result not recorded.** D1 and Telegram do not share one transaction. Blind repetition can duplicate a customer message, topic, or manager reply; preserve an unknown result and reconcile that side effect.
3. **Telegram confirmed failure.** Record the failure and notify the affected side when possible. A failed notice needs its own diagnostic result.
4. **Webhook payload no longer exists.** Without stored content, the backend cannot recreate a missing customer message. The contract must either require a resend or define temporary content storage.

## Recommended minimal Prototype assumption

**Recommendation only; not accepted behavior:** store routing state and attempt metadata, but no full conversation transcript. For an inbound post with an unknown result, do not replay automatically. Tell the customer that transfer is unconfirmed and may require resending; record failure of that notice for safe diagnostics.

If eventual delivery of the original message without customer action is required, temporary content storage needs explicit retention and access rules. A delivery-attempt record alone is insufficient.

The existing protected operational API in #6 covers uncertain topic creation. Extending it to message-delivery history or recovery is a separate scope choice; #9 currently excludes an admin delivery-history API.

## Findings for MVP and later

### G7: integration lifecycle

- **Observation:** registration and initial readiness are defined; disabling support, rotating a bot token, disconnecting Telegram Business, and replacing a group are not.
- **Recommendation:** decide what happens to webhooks, pending attempts, existing topic mappings, and customer-visible status for each operation, including which changes require a new mapping.
- **Placement:** before grooming the relevant MVP integration-management Issues; then record accepted behavior in SPEC and Decisions.
- **Risk if skipped:** lifecycle changes can orphan webhooks, credentials, mappings, or unresolved deliveries.

### G8: product access in Shell

- **Observation:** Prototype uses one operator credential. The reference delegates administrator validation to its own upstream backend, which is not this project's contract.
- **Recommendation:** define the authoritative Shell product or remote ID and who can read or change each integration. Enforce product scope on every relevant API action.
- **Placement:** extend Open Question 10 before grooming `MVP-07`–`MVP-10`.
- **Risk if skipped:** Shell integration can expose or mutate another product's Telegram configuration.

### G9: work visibility

- **Observation:** the reference inbox exposes whether a thread waits for a reply; current managers work directly in Telegram topics.
- **Recommendation:** first decide whether managers need a separate waiting-for-support view or counts, and which message and delivery outcomes change that state.
- **Placement:** optional product discovery after Prototype. Add a User Story and Issue only if chosen.
- **Risk if skipped:** none for the current Prototype; implementing it without a product need would add state and retention scope.

## Possible later extensions

| Idea | Possible value | Decision required first |
| --- | --- | --- |
| Aggregate workload metrics | Count unanswered conversations and response time by product without necessarily storing transcripts. | Define what counts as a reply, including failed and unknown sends. |
| Searchable support history | Search or review conversations outside Telegram. | Store message contents and define retention, product access, media, and deletion. |
| AI drafts or summaries | Help managers with repeated questions. | Approve the product goal, data access, cost, manager review, and separation of internal and customer-visible information. |
| Read-only cross-system support API | Feed a later authorized support workspace. | Define authentication and product-scoped authorization. |

These ideas do not justify adding a support inbox, Durable Object stream, Queue, R2, or Workers AI to the Prototype. The reference has a web widget and workflows that this product does not require.

## Evidence cautions

- The reference's architecture binding list does not list the Durable Object described as required in its Cloudflare handoff. Check source and deployment configuration before borrowing that fact.
- Its credentials audit and troubleshooting guide use different admin-auth examples. Their authentication adapter does not establish the future Shell contract here.
- Its documentation flags exposed debug routes and secrets stored as plain variables as operational risks. Borrow the diagnostic questions, not those routes or credentials.
- Cloudflare limits and prices can change. Check current official Workers limits and D1 pricing during #11.
- Telegram webhook retries do not create a transaction with D1 or downstream Telegram calls.

Reference material:

- [web-chat-backend Operations](https://github.com/AA-asomarket/web-chat-backend/blob/dev/docs/OPERATIONS.md)
- [web-chat-backend Troubleshooting](https://github.com/AA-asomarket/web-chat-backend/blob/dev/docs/TROUBLESHOOTING.md)
- [web-chat-backend Architecture](https://github.com/AA-asomarket/web-chat-backend/blob/dev/docs/ARCHITECTURE.md)
- [web-chat-backend Cloudflare handoff](https://github.com/AA-asomarket/web-chat-backend/blob/dev/README_HANDOFF_CLOUDFLARE.md)
- [web-chat-backend credentials audit](https://github.com/AA-asomarket/web-chat-backend/blob/dev/docs/CREDENTIALS_AUDIT.md)
- [web-chat-backend MCP read API](https://github.com/AA-asomarket/web-chat-backend/blob/dev/docs/MCP_READ_API.md)
- [Cloudflare Workers limits](https://developers.cloudflare.com/workers/platform/limits/)
- [Cloudflare D1 pricing](https://developers.cloudflare.com/d1/platform/pricing/)
- [Telegram Bot API setWebhook](https://core.telegram.org/bots/api#setwebhook)
