# Work process

- GitHub Issues are the active backlog. The current active stage is `prototype`.
- Work on one open issue at a time, through the roles and lifecycle below.

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

Active stage label: `prototype`.

An issue is eligible when it is open, carries that label, and every dependency listed in it is closed. If several are eligible, take the lowest issue number. If it is blocked or conflicts with `_docs/decisions.md`, stop and report it. Stop when none remain.

1. Select the next eligible issue.
2. Delegate grooming to `pm`. Accept the issue for implementation only after its Goal, acceptance criteria, Out of scope, Constraints, and verification checks are concrete.
3. Delegate the groomed issue to `engineer`. On a retry, include the QA report. The engineer leaves the issue open.
4. Delegate verification to `qa`.
5. On `FAIL`, return to step 3 with that report.
6. On `PASS`, the acceptance criteria are checked and the issue is closed.
7. Repeat from step 1.

Do not skip grooming. The engineer does not close the issue. QA does not change application code.

## Reference

- Issue references `P-01` through `P-19` point to the [archived Prototype plan](archived/plan.md). Archived plans, specifications, architecture, and task drafts are reference material. Current decisions and GitHub Issues take precedence if they differ.
- When work moves to MVP, use accepted decisions, open MVP questions, and archived Prototype material to draft that stage's scope and issues. Keep `_docs/tech-stack.md` current; write a new architecture document only if the stage needs a cross-issue design.
