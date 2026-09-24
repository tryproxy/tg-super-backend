# Work process

GitHub Issues are the active backlog. Active stage: `prototype`. Work on one implementation Issue at a time.

## Selection and delegation

The main session coordinates work using [routing.md](routing.md). It supplies the Issue, repository path, active stage, and any previous role report. Models and effort belong to the agent profiles.

Select an open Issue with the active stage label and all declared dependencies closed. Exclude `finding` tracking Issues. Continue the current Issue while it remains eligible; otherwise select the lowest Issue number.

| Issue state | Next role |
| --- | --- |
| No handoff label | [PM](team/pm.md), profile `pm` |
| `status:ready-for-implementation` | [Engineer](team/engineer.md), profile from Routing |
| `status:ready-for-qa` | [QA](team/qa.md), profile from Routing |
| Closed | Excluded |
| Both handoff labels | Stop and report inconsistent state |

Role files own their actions, reports, and outgoing label changes.

## Lifecycle

1. Select the Issue and check its latest role report for an unanswered request. If one exists, wait for the user's answer instead of restarting the role.
2. Choose the next role from the table and apply Routing. Delegate with only the context required by that role; include the QA report for an Engineer retry.
3. Read the returned report. Apply any exact GitHub update returned because the role could not write; do not repeat its work. If the main session also cannot write, preserve the update and stop. Do not advance on an unapplied handoff.
4. If Engineer reports that the contract cannot be implemented, remove the implementation label and return the concrete obstacle to PM for grooming. This transition belongs to the main session.
5. If PM needs a user decision, record the question on the Issue and end this run. Resume only after an explicit answer; absence of labels is not permission to repeat the question.
6. Otherwise refresh the Issue state and repeat. Stop when no eligible Issue remains.

A routing approval request also ends the run until answered. If an Issue is otherwise unable to proceed, report the concrete obstacle and stop.

After QA, the main session checks linked Finding completion conditions against the owning Issues' QA evidence. When all conditions are met, close the tracking Issue and archive its record according to [Research workflow](research/README.md).
