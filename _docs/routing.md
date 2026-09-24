# Issue routing

Routing labels describe the capability or workload required by an Issue. They do not name a model vendor.

## Responsibilities

- PM assigns `reasoning` and `workload` during grooming when they apply and records one short reason for each label.
- The main session selects the Engineer and QA profiles from `reasoning`. The `workload` label does not select a profile.
- Agent descriptions do not override these rules. If the labels are wrong, return the Issue to PM for reclassification before delegation.

## Labels

- `reasoning` — the Issue requires difficult design, recovery, concurrency, or debugging.
- `workload` — implementation or verification has a large footprint in code, context, integrations, or runtime. File count alone is not enough.

An Issue carrying both `reasoning` and `workload` requires an explicit routing decision from the user before a reasoning profile is delegated. The main session stops and asks whether to use the higher-cost reasoning route for that Issue. Silence is not approval.

When approved, the main session records the routing decision on the Issue. It covers both Engineer and QA while the Issue scope and routing labels remain unchanged.

If the user wants decomposition, they explicitly return the Issue to PM and define that assignment. PM does not split the Issue or create sub-issues merely because both labels are present.

## Profile selection

| Issue labels | Engineer profile | QA profile |
| --- | --- | --- |
| Without `reasoning` | `engineer` | `qa` |
| With `reasoning` | `engineer-reasoning` | `qa-reasoning` |

Routing labels do not change dependency order, stage priority, or Issue eligibility.
