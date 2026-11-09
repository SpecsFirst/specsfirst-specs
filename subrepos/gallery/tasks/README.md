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
| TASK-004 | Votes and publication workflow | done | TASK-002, TASK-003 |
| TASK-005 | One-page contributing guide | done | TASK-001 |
| TASK-006 | Render the file tree like `tree` | done | — |
| TASK-007 | Redesign the site after the mockups | done | TASK-006, TASK-008 |
| TASK-008 | Create each template's Discussion in the publish workflow | done | — |
| TASK-009 | Submit a template through an issue form | done | — |
| TASK-010 | Where specs live, and the install command | done | — |
| TASK-011 | Agnostic templates only, placement and command in the submission | in_progress | TASK-010 |

## Backlog

Planned tasks, one line each. The full `TASK-*.md` is written when the task is
about to start, so its contract uses the actual outputs of its dependencies.

- `submit.count_tree` repeats the counting of `tree.render_tree` (TASK-006, written in
  parallel): use `render_tree(...).summary` once both are merged.
- More than 2000 Discussions would fail the publish (`MAX_PAGES` in `votes.py`).
- Drop `markdown-it-py` completely: `tests/test_submit.py` is its last user (checks that
  comments render inert); replace that check or keep the dev dependency on purpose.
- Drop `preview` from detect and `detected.json`: the site no longer shows it (one less blob
  fetched per template). `tools` stays: since TASK-011 it lists tool-specific files.

Order: TASK-010, then TASK-011 on its branch; one pull request for both (operator: one PR
for the gallery changes).

## Transient exchange

Intermediate exchange between orchestrator and coders goes in
`tasks/.exchange/` — **git-ignored**, not part of the durable record. What a task
teaches (decisions, findings) goes into `../main.md`.
