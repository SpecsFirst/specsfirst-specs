---
id: TASK-010
title: Where specs live, and the install command
status: open             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/gallery/main.md
subrepo: gallery
issue:
assignee: coder
model: albert/deepseek-v4-flash-0731
created: 2026-11-09
updated: 2026-11-09
---

# TASK-010: Where specs live, and the install command

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

Show where a template's specs live (their own repo or the code repo), let visitors filter on
it, and show an install command that comes from the entry or, failing that, from Copier.

## Context

Read `subrepos/gallery/main.md` (Scope, Decisions of 2026-11-09, Design tokens) and
`architecture.md` (ADR-001, ADR-007: no inline script or style attribute).

The site was just redesigned (TASK-007). **Do not change the design**: add the new elements
in the same style, with the existing tokens, classes and patterns (the tag filter's
`aria-pressed` buttons, the `<dl class="meta">` rows, the `.cmd` block).

Operator brief (2026-11-09), translated:

- **Placement.** Each entry may declare `placement: standalone | in-repo`. It is optional in
  the schema and defaults to `standalone` (operator's choice: the two existing entries are
  standalone and are not edited). The submission form always asks for it (TASK-011).
- **"Lives in"** metadata row, second, right after License, on the cards and on the template
  page: `Own repo` for standalone, `Code repo` for in-repo.
- **Placement filter**, above the tags: a segmented control `All · Own repo · Code repo`.
  It combines with the tag filter (a card is shown when it matches both) and updates
  "N templates, sorted by stars". No `standalone` or `in-repo` tag is created.
- **Intro.** Replace the home page paragraph with, exactly: "Specs hold what a project is
  about (goals, conventions, tasks), written for the humans and AI agents who build it, in
  their own repo or next to the code. Compare the structures below, star the ones you like,
  and start yours with a single command."
- **Command.** An optional `command` field in the entry is shown as is. Otherwise, when
  detect found a `copier.yml`: `copier copy <url> my-project` for standalone,
  `copier copy <url> .` for in-repo. Otherwise the command block is hidden.
- **Copy button**: a small transparent square at the right of the block,
  `aria-label="Copy command"`, which turns into ✓ for 1.5 s. It already exists
  (`site/template.html`, `site/assets/site.js`): check it matches, change it only if it
  does not.

## Input contract

```yaml
repo: /home/omni/specfirst/gallery
worktree: /home/omni/specfirst/.worktrees/gallery-placement
branch: task/010-placement
depends_on: []
inputs:
  - schema/template.schema.json
  - src/gallery/validate.py
  - src/gallery/build.py
  - site/
  - CONTRIBUTING.md
scope:
  write:
    - schema/template.schema.json   # add `placement` and `command`, both optional
    - src/gallery/validate.py       # extra checks on `command`, if the schema cannot express them
    - src/gallery/build.py
    - site/index.html
    - site/template.html
    - site/_parts.html
    - site/assets/site.css
    - site/assets/site.js
    - tests/test_build.py
    - tests/test_validate.py
    - CONTRIBUTING.md               # document the two fields in "Without the form" only
  read_only:
    - templates/                    # never edited: the operator aligns the entries
    - taxonomy.yml
    - src/gallery/registry.py
    - src/gallery/detect.py
    - src/gallery/submit.py         # TASK-011 changes the submission
    - .github/
    - tests/helpers.py
checks:
  - uv sync
  - uv run pytest
  - uv run ruff check .
  - uv run python -m gallery.validate
  - build the site from fixtures (one standalone entry with copier, one in-repo entry with
    copier, one entry with `command`, one with neither) and read the generated HTML
```

## Output contract

```yaml
artifacts:
  - schema, build, site templates, CSS, JS and tests above
contract:
  - Schema: `placement` is an enum [standalone, in-repo] with default standalone; `command` is
    a string of 1 to 300 characters on one line. Both optional; `required` is unchanged.
    Validation refuses a `command` with a control character (it is pasted into terminals).
  - Build: a missing `placement` is standalone. The view has the placement and its label.
    `command` from the entry wins and is shown even when detect failed; otherwise
    `copier_command` keeps its `ref` and `path` variants, with `.` as the destination for
    in-repo (the `path` variant clones into `../template-src` for in-repo, so nothing lands
    in the code repo). No copier and no `command`: no block.
  - Every visible string comes from Jinja autoescaping; nothing from an entry reaches a
    `style` attribute, a script, or an unquoted attribute.
  - "Lives in" is the second row of `<dl class="meta">`, on cards and on the template page.
  - The placement control is a group of three `<button aria-pressed>` with a visible group
    label or `aria-label="Filter by location"`, hidden without JavaScript like the tag
    buttons (every card visible without JS). It is always shown when there are cards.
    Selecting a placement and a tag shows the cards matching both; the count text follows.
  - The intro paragraph is the exact sentence of the brief.
  - Layout: at 390 px wide, no horizontal page scroll (the segmented control fits or wraps,
    the command line scrolls inside its own block). Contrast and target sizes keep the
    main.md principles (≥ 4.5:1, ≥ 40 px targets).
  - Tests cover: default placement, both labels, the three command cases and the in-repo
    destination, the meta row order, the filter markup, and the schema/validation of the
    two fields.
  - No entry under templates/ is added, removed or edited.
returns: the branch with its commit(s), and the filled Result section
```

## Attempts

> Filled in by the orchestrator, one line per (re)launch.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

### Artifacts produced

### Opinion / concerns

### CLARIFICATION NEEDED
