---
id: TASK-001
title: Organization profile page
status: done             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/.github/main.md
subrepo: .github
issue:
assignee: coder
model: anthropic/claude-sonnet-5-5
created: 2026-11-06
updated: 2026-11-06
---

# TASK-001: Organization profile page

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

Write `profile/README.md`, the page GitHub shows on the SpecsFirst organization.

## Context

Read `main.md` and `subrepos/.github/main.md`. For the facts about the gallery,
read `subrepos/gallery/main.md`. These specs are the only source of facts about
SpecsFirst.

## Input contract

```yaml
repo: /home/omni/specfirst/.github
worktree: /home/omni/specfirst/.worktrees/dotgithub-org-profile
branch: task/001-org-profile
depends_on: []
inputs:
  - subrepos/.github/main.md
  - subrepos/gallery/main.md
  - /home/omni/specfirst/gallery/schema/template.schema.json
  - /home/omni/specfirst/gallery/templates/dev-studio-specs-template.yml
scope:
  write:
    - profile/README.md
  read_only: []
checks:
  - test -f profile/README.md
  - test "$(wc -l < profile/README.md)" -le 40
```

## Output contract

```yaml
artifacts:
  - profile/README.md
contract:
  - In English, 40 lines at most, plain Markdown that renders on GitHub.
  - Says in one or two sentences what SpecsFirst is, from the global goal.
  - Links to the gallery https://specsfirst.github.io/gallery/ as the main call to
    action.
  - Shows that sharing a template is trivial, with a real five-line entry as a YAML
    block, taken from a real entry and valid against the schema.
  - Links to https://github.com/SpecsFirst/gallery/blob/main/CONTRIBUTING.md and to
    the repo https://github.com/SpecsFirst/gallery.
  - States no number of templates or users, no roadmap, no invented feature, no
    badges, no images, and no link to a page that the specs do not mention.
returns: the branch with its commit, and the filled Result section
```

## Attempts

> Filled in by the orchestrator, one line per (re)launch.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

Wrote `profile/README.md` (30 lines): one-line definition from the global goal,
gallery link as main call to action, the real entry `dev-studio-specs-template.yml`
verbatim as a 5-line YAML block (validated against `template.schema.json` with
jsonschema, and byte-identical to the entry file), links to CONTRIBUTING.md and the
gallery repo. Checks pass (file exists, 30 <= 40 lines). Gallery feature claims
(filters, sort by votes/update, vote/comment, any forge, auto-detection) come
from `subrepos/gallery/main.md`. Branch `task/001-org-profile`, one commit
`docs: add organization profile README`.

### Artifacts produced

- `profile/README.md` (in worktree `/home/omni/specfirst/.worktrees/dotgithub-org-profile`)

### Opinion / concerns

- The profile links to `CONTRIBUTING.md` and the gallery site, which are produced by
  `gallery` tasks: the links are dead until those land.
- The sample entry points to a gitlab.inria.fr repo (as required: real entry); fine,
  and it shows the "any forge" point.

### CLARIFICATION NEEDED
