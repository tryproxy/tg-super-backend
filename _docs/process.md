# Work process

- GitHub Issues are the active backlog. The current active stage is `prototype`.
- Work on one open issue at a time. An issue is eligible when it carries the active stage label and all dependencies listed in it are closed. If several issues are eligible, select the lowest issue number.
- If the selected issue is blocked or conflicts with `_docs/decisions.md`, stop and report the blocker before grooming or implementation.

## Grooming

- Treat a selected issue as not ready for implementation until it has been groomed.
- Read the issue's Goal, acceptance criteria, Out of scope, Constraints, and dependencies. Read `_docs/decisions.md` and `_docs/tech-stack.md`; consult `_docs/open-questions.md` when unresolved behavior affects the issue.
- Inspect the current repository state and confirm that the issue describes one coherent outcome.
- Make every acceptance criterion concrete and verifiable, identify the focused tests or observable checks required, and confirm the scope, constraints, and dependencies.
- Amend the issue before coding when a material decision, acceptance criterion, dependency, or verification requirement is missing. A decision that affects multiple issues belongs in `_docs/decisions.md`.
- If grooming reveals a missing product decision, stop and report it rather than inventing behavior during implementation.

## Implementation and verification

- Treat the groomed issue as the implementation contract. Implement the smallest change that meets it and do not expand the issue's scope.
- When behavior changes, update the relevant decisions and issue documentation in the same change.
- Commit at coherent, verified checkpoints. A small issue may need only one focused commit; do not accumulate unrelated work.
- Before closing, reread the groomed issue and verify every acceptance criterion with its focused test or observable check.
- Record concrete verification evidence in the issue, check each satisfied acceptance criterion, and close the issue only after all criteria are met and verified.
- If the backlog is being processed as a batch, repeat from issue selection after closeout. Otherwise stop after the selected issue.

- Issue references `P-01` through `P-19` point to the [archived Prototype plan](archived/plan.md). Archived plans, specifications, architecture, and task drafts are reference material. Current decisions and GitHub Issues take precedence if they differ.
- When work moves to MVP, use accepted decisions, open MVP questions, and archived Prototype material to draft that stage's scope and issues. Keep `_docs/tech-stack.md` current; write a new architecture document only if the stage needs a cross-issue design.
