# Research workflow

Research captures discoveries made after the active backlog was created. It supplies evidence and ordering for grooming; it does not replace User Stories, SPEC, Decisions, Open Questions, or an Issue contract.

## Finding

A **Finding** is a late-discovered improvement, constraint, or risk whose omission could cause material rework, break dependency order, or leave meaningful behavior, failure handling, or operations undefined.

A Finding is not automatically:

- an accepted product requirement;
- an implementation Issue;
- a blocker;
- a reason to reopen completed work.

Keep the stable `G` identifier after a Finding is renamed, scheduled, implemented, or archived.

## Lifecycle

| Status | Meaning | Effect on current work |
| --- | --- | --- |
| `proposed` | Evidence and a recommendation are recorded, but no disposition is accepted. | Does not delay implementation. |
| `accepted` | The Finding must be addressed at its recorded stage and placement. Its product answer may still remain open until scheduled grooming. | Blocks only when placed before the current Issue; otherwise follow its recorded placement. |
| `postponed` | The Finding remains useful but is intentionally deferred. | Does not expand or delay the current stage. |
| `rejected` | The Finding will not be adopted; the reason is recorded. | No implementation effect. |
| `implemented` | The accepted result and owning implementation have passed QA. | Move the record to `archived/` with its evidence and final links. |

The user accepts, postpones, or rejects a Finding. Research recommendations alone never change its status.

## Placement

| Placement | Meaning |
| --- | --- |
| Before an Issue | Resolve the open choice and reconcile active requirements before handoff. |
| Inside an Issue | Add the accepted behavior, constraint, or verification to that Issue during grooming. |
| After an Issue | Keep it scheduled for later work without delaying the current Issue. |
| Separate Issue | Create linked implementation work when the Finding cannot be owned safely by an existing Issue. |

Impact and placement are separate. A Finding may affect product behavior, implementation or architecture, or diagnostics and recovery; that impact does not determine where it belongs in the queue.

## Required record

Each Finding records:

- a stable ID and short name;
- source and evidence;
- current status and affected stage;
- affected Issues and dependencies;
- impact and placement;
- likely rework or failure if skipped;
- recommendation and any unresolved choice;
- completion conditions and implementation home.

Use [findings.md](findings.md) as the active registry and schedule. When implementation and QA are complete, preserve the final decision and evidence under [archived](archived/). Postponed Findings stay in the active registry so their dependency chain remains visible.
