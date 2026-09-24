# Tech stack

Serverless operation is the priority. Prototype choices are approved; MVP and Release entries below are drafts, not approved dependencies or cost commitments. Draft choices require approval for their stage before implementation. Component boundaries are in [architecture](architecture.md).

## Prototype — approved

| Purpose | Choice | Prototype cost |
| --- | --- | --- |
| Language and local tooling | TypeScript, Node.js 22, pnpm 11 | Free |
| HTTP and admin API | Hono | Free, open source |
| Telegram Bot API | grammY | Free, open source |
| Hosting | Cloudflare Workers Free | Free within plan limits |
| Database | Cloudflare D1 Free | Free within plan limits |
| Input validation | Zod | Free, open source |
| Local development and deployment | Wrangler | Free tooling |
| Tests | Vitest | Free, open source |

## MVP — draft

| Purpose | Proposed choice | Status |
| --- | --- | --- |
| Telegram Business | Reuse [grammY](https://grammy.dev/advanced/business) and the Bot API in this backend; add one shared connector bot identity | No new Telegram framework proposed |
| Shell setup | Existing Runtime MF Shell calls this backend API, implemented with Hono | No new backend framework proposed |
| Manager CSV and invitations | Reuse the Worker and D1; choose a CSV parser after the format is agreed | CSV format remains open |

## Release — draft

| Purpose | Candidate | Status |
| --- | --- | --- |
| MTProto group provisioning and direct invitations | Separate TypeScript component using [teleproto](https://github.com/sanyok12345/teleproto) | Candidate only; verify QR login, required MTProto methods, and runtime compatibility before approval |
| Execution and authorized-session storage | Prefer an on-demand serverless runtime; choose the host and protected storage after a compatibility check | Open; see [Open Question 12](open-questions.md#release) |

The message-routing Worker remains separate from MTProto provisioning. Recheck free-tier fit for MVP and Release before approving their stack; no new provider or paid service is selected here.
