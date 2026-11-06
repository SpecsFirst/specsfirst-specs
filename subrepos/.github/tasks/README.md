# Tasks

Index and global status of all tasks. Each task is one file:
`TASK-<NNN>-<slug>.md`.

## Status legend

- **open** — defined, not started
- **in_progress** — being executed
- **blocked** — stopped by the executor (reason in the task's Result)
- **done** — completed and contract satisfied
- **needs_review** — done, awaiting human/orchestrator review
- **rejected** — closed without completion (reason in the task)

## Board

| Task | Title | Status | Depends on |
|------|-------|--------|------------|
| TASK-001 | Organization profile page | in_progress | — |

## Transient exchange

Intermediate exchange between orchestrator and coders goes in
`tasks/.exchange/` — **git-ignored**, not part of the durable record. What a task
teaches (decisions, findings) goes into `../main.md`.
