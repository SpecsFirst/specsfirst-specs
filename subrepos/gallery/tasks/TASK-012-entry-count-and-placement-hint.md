---
id: TASK-012
title: Stop pinning the entry count in tests, clarify the placement question
status: open             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/gallery/main.md
subrepo: gallery
issue:
assignee: orchestrator
model: claude-opus-5-5
created: 2026-11-10
updated: 2026-11-10
---

# TASK-012: Stop pinning the entry count in tests, clarify the placement question

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

A submission pull request that adds a valid entry passes CI, and the form's placement
question is read the same way by every submitter.

## Context

First real submissions (issues #13 and #15, pull requests #14 and #16) failed CI:
`tests/test_registry.py::test_load_registry_uses_real_files` asserts
`len(registry.entries) == 2`, so any new entry breaks it. Its intent is "every real file
loads".

Issue #13 (an in-repo template) answered "In their own repo": the hint "Does the
repository hold the specs themselves, or just the code they drive?" is about the template
repository, which holds specs in both cases. The question is where the specs sit in a
project made from the template. The two choices stay as they are (operator's brief).

Small enough for the orchestrator to do directly.

## Input contract

```yaml
repo: /home/omni/specfirst/gallery
worktree: /home/omni/specfirst/.worktrees/gallery-entry-count
branch: task/012-entry-count-and-placement-hint
depends_on: []
scope:
  write:
    - tests/test_registry.py
    - .github/ISSUE_TEMPLATE/submit-template.yml   # placement description only
  read_only:
    - templates/
    - src/
checks:
  - uv run pytest
  - uv run ruff check .
  - uv run --with check-jsonschema check-jsonschema --builtin-schema vendor.github-issue-forms .github/ISSUE_TEMPLATE/submit-template.yml
```

## Output contract

```yaml
contract:
  - The test compares the loaded entries with the `templates/*.yml` files on disk, not
    with a fixed number. Adding or removing an entry never breaks the suite.
  - The placement description asks where the specs sit in a project made from the
    template; labels and choices unchanged.
returns: the branch with its commit(s), and the filled Result section
```

## Attempts

> Filled in by the orchestrator, one line per (re)launch.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.
