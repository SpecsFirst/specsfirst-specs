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
| TASK-001 | Registry and validation Action | done | — |
| TASK-002 | Detect template information safely | done | TASK-001 |
| TASK-003 | Static site | done | TASK-001 |
| TASK-004 | Votes and publication workflow | in_progress | TASK-002, TASK-003 |
| TASK-005 | One-page contributing guide | done | TASK-001 |

## Backlog

Planned tasks, one line each. The full `TASK-*.md` is written when the task is
about to start, so its contract uses the actual outputs of its dependencies.

_None._

## Transient exchange

Intermediate exchange between orchestrator and coders goes in
`tasks/.exchange/` — **git-ignored**, not part of the durable record. What a task
teaches (decisions, findings) goes into `../main.md`.
