# Work process

- GitHub Issues are the active backlog. The current active stage is `prototype`.
- User Stories state user goals; `SPEC.md` defines observable behavior; Decisions explain accepted choices; Issues divide that work into tasks.
- Work on one open issue at a time, through the roles and lifecycle below.
- [Research Findings](research/findings.md) records late discoveries and their position in the Issue queue. A Finding does not change accepted requirements by itself.

## Roles

- The main session selects the Issue, determines the next role from its handoff status, selects the project agent profile through the routing rules, delegates the work, and checks the handoff. It does not groom, implement, or verify the Issue itself.
- PM grooms and classifies one Issue and follows `_docs/team/pm.md`.
- Engineer implements one groomed Issue and follows `_docs/team/engineer.md`.
- QA verifies one completed Issue and follows `_docs/team/qa.md`.

## Delegation

- PM uses the `pm` project agent. Engineer and QA use the project agent profile selected for the Issue.
- Model and effort settings belong to the agent profile.
- If GitHub denies a role write access or is unavailable, the role returns the exact proposed update. The main session reviews that update and applies it. It does not redo the role's work.

## Routing

Use [Issue routing](routing.md) for label ownership, classification, and project agent profile selection.
A routing decision required there pauses the lifecycle until the user answers.

## Handoff labels

Two mutually exclusive handoff labels persist the current lifecycle state on GitHub so another session or harness can resume the Issue:

- `status:ready-for-implementation` — PM finished grooming, no blocking question remains, and every declared dependency is closed.
- `status:ready-for-qa` — Engineer committed the implementation and posted the verification evidence required by the Issue.

An Issue carries neither status while it needs PM work or is blocked. Finding tracking Issues use `finding` and their routing classification, but do not receive either handoff status because implementation remains in the owning stage Issue.

## Lifecycle

Active stage label: `prototype`. Finding tracking Issues are outside the implementation queue and are closed by the main session when their stated conditions and owning implementation Issues are verified.

An implementation Issue is eligible when it is open, carries the active stage label, and every dependency listed in it is closed. If several are eligible for the same next role, take the lowest Issue number. Stop when none remain. If the selected Issue is otherwise blocked, report it.

1. Select the next eligible Issue. Its handoff label determines the next role: no handoff label means PM, `status:ready-for-implementation` means Engineer, and `status:ready-for-qa` means QA.
2. Delegate grooming to `pm` when no handoff label is present. PM follows `_docs/team/pm.md`. A blocked Issue remains without a handoff label; a completed grooming handoff adds `status:ready-for-implementation`.
3. Delegate an Issue carrying `status:ready-for-implementation` and its linked SPEC requirements to the selected Engineer profile. On a retry, include the QA report. The Engineer leaves the Issue open and replaces the implementation status with `status:ready-for-qa` only after committing verified work and posting evidence. If the Issue cannot be completed without changing its contract, Engineer reports the concrete obstacle and stops; the main session removes the implementation status and returns the Issue to PM.
4. Delegate an Issue carrying `status:ready-for-qa` to the selected QA profile.
5. On `FAIL`, QA replaces `status:ready-for-qa` with `status:ready-for-implementation` and returns the verification report to Engineer.
6. On `PASS`, QA marks every acceptance-criterion checkbox `[x]`, removes `status:ready-for-qa`, and closes the Issue.
7. Repeat from step 1.

Do not skip grooming. The engineer does not close the issue. QA does not change application code.
