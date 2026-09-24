# Work process

- GitHub Issues are the active backlog. The current active stage is `prototype`.
- User Stories state user goals; `SPEC.md` defines observable behavior; Decisions explain accepted choices; Issues divide that work into tasks.
- Work on one open issue at a time, through the roles and lifecycle below.
- [Gap Implementation Roadmap](research/gap-implementation-roadmap.md) schedules research findings against Issues; [Gap Analysis](research/gap-analysis.md) explains them. Neither document changes accepted requirements by itself.

## Roles

- The main session selects the issue, delegates each role, and checks the handoff. It does not groom, implement, or verify the issue itself.
- `pm` grooms one issue and follows `_docs/team/pm.md`.
- `engineer` implements one groomed issue and follows `_docs/team/engineer.md`.
- `qa` verifies one completed issue and follows `_docs/team/qa.md`.

## Delegation

- Invoke the project agent whose name is the role: `pm`, `engineer`, or `qa`.
- The session resolves that name through its own subagent mechanism. Model, effort, and tool settings stay in the agent file.
- If a role cannot write to GitHub, it returns the exact proposed update. The main session reviews that update and applies it. It does not redo the role's work.

## Lifecycle

Active stage label: `prototype`. Gap tracking Issues carry only `gap`, are outside this implementation queue, and are closed by the main session when their stated conditions and owning implementation Issues are verified.

An issue is eligible when it is open, carries that label, and every dependency listed in it is closed. If several are eligible, take the lowest issue number. Stop when none remain. If the selected issue is otherwise blocked, report it.

1. Select the next eligible issue.
2. Delegate grooming to `pm`.
   - PM checks the issue against its User Story, when applicable, and relevant SPEC requirements, and checks the Gap Implementation Roadmap for findings scheduled at this Issue. PM reads only the relevant Gap Analysis entries.
   - If only the issue is outdated, PM corrects it. For an unclear choice or conflict, PM consults Decisions, then Open Questions if needed. Gap research provides evidence and options, not an overriding answer.
   - If a scheduled gap changes behavior, PM reports the exact question and recommendation. Once the choice is accepted, reconcile SPEC, Decisions or Open Questions, and the issue before handoff. A gap that affects only implementation detail can be included in the issue when accepted requirements already determine the behavior.
   - If PM discovers a new gap, first check whether active requirements already cover it. For a real gap, record evidence and the affected Issue in Gap Analysis, then place it before or inside that Issue in the Roadmap. A gap blocking the current Issue is reported with options and prevents handoff; a future gap is recorded without delaying current work. If the affected Issue is already closed, propose a follow-up Issue; reopen the old one only if its accepted criteria were not actually met.
   - Hand off only after current-issue discrepancies are resolved and Goal, acceptance criteria, Out of scope, Constraints, and verification checks are concrete.
3. Delegate the groomed issue and its linked SPEC requirements to `engineer`. On a retry, include the QA report. The engineer leaves the issue open.
4. Delegate verification of the issue and its linked SPEC requirements to `qa`.
5. On `FAIL`, return to step 3 with that report.
6. On `PASS`, the acceptance criteria are checked and the issue is closed.
7. Repeat from step 1.

Do not skip grooming. The engineer does not close the issue. QA does not change application code.

## Reference

- Current requirements are in [SPEC.md](../SPEC.md), with goals in [User Stories](user-stories.md) and accepted choices in [Decisions](decisions.md). Reconcile disagreements among active documents before implementation. The research files schedule and explain potential changes; they are not additional acceptance criteria until reconciled with the active documents and Issue.
