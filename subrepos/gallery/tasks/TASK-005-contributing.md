---
id: TASK-005
title: One-page contributing guide
status: done             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/gallery/main.md
subrepo: gallery
issue:
assignee: coder
model: anthropic/claude-sonnet-5-5
created: 2026-11-06
updated: 2026-11-06
---

# TASK-005: One-page contributing guide

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

Add a one-page `CONTRIBUTING.md` that lets someone submit a template without reading anything
else. The site and the organization profile already link to it.

## Context

Read `subrepos/gallery/main.md` (Scope, Evolution rules, Decisions) and ADR-002 in
`architecture.md`. Check every rule you state against `schema/template.schema.json`,
`taxonomy.yml` and `src/gallery/validate.py`: the code decides, not this spec.

Votes and the publish workflow are being added at the same time by TASK-004. Rely on what is
stated below about them, and do not read or wait for that branch.

## Input contract

```yaml
repo: /home/omni/specfirst/gallery
worktree: /home/omni/specfirst/.worktrees/gallery-contributing
branch: task/005-contributing
depends_on: [TASK-001]
inputs:
  - schema/template.schema.json
  - taxonomy.yml
  - templates/
  - src/gallery/registry.py
  - src/gallery/validate.py
scope:
  write:
    - CONTRIBUTING.md
  read_only:
    - everything else
checks:
  - uv sync
  - uv run python -m gallery.validate
  - the example of the guide, copied as a real entry in a scratch copy of the repo, passes
    `uv run python -m gallery.validate --root <scratch copy>`
```

## Output contract

```yaml
artifacts:
  - CONTRIBUTING.md
contract:
  - One page: at most 80 lines, in English, in plain sentences for someone who has never used
    the repo. Lead with the steps of a submission.
  - Submitting: a pull request adding one file `templates/<id>.yml`. It states the id rule
    (from the file name, lowercase letters and digits with single hyphens, used in URLs and
    Discussion titles), and gives a complete example entry that passes validation.
  - Fields: a short table of the required and optional fields with their real limits.
  - It states the kinds and tags come from `taxonomy.yml` and links to that file. It does not
    copy their lists, which would go stale. A new tag is a pull request on `taxonomy.yml`
    saying which templates need it; tags are deprecated, never deleted or renamed.
  - It lists what the gallery detects on its own, so nobody declares it: license, `copier.yml`,
    tool-specific files, last update, file tree, `main.md` preview. Having a `main.md` and a
    license file at the template's root is worth a sentence, since they drive the page.
  - What a template must be: a public git repository reachable over https, on any forge.
    The gallery only reads its git objects and never runs or checks out its content. If it
    cannot be fetched, the page still shows the entry's fields.
  - After the merge: the site is rebuilt on each push to main and every six hours. Votes are
    👍 reactions on the template's page, through giscus (a GitHub sign-in), and they refresh
    with each rebuild.
  - Running the check locally: `uv sync` then `uv run python -m gallery.validate`. A pull
    request runs the same check automatically, and a maintainer merges it.
  - No section on code contribution beyond one line naming `uv run pytest` and
    `uv run ruff check .`. No badges and no code of conduct.
returns: the branch with its commit(s), and the filled Result section
```

## Attempts

> Filled in by the orchestrator, one line per (re)launch.

- 2026-11-06 #1: done. Review replayed the guide's example through the validator (exit 0), a file
  with nine errors (all reported at once), and read each limit against the schema and validator.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

`CONTRIBUTING.md` added (75 lines, English), commit `263ab1a` on `task/005-contributing`.
Sections: steps first, id rule, complete example, fields table with the real limits,
taxonomy link and tag evolution, what a template must be, what is detected (plus the
`main.md` and license sentence), after the merge (rebuild on push and every six hours,
👍 through giscus), local check, one line for `uv run pytest` / `uv run ruff check .`.

Checks run, real results:
- `uv sync`: ok.
- `uv run python -m gallery.validate` in the worktree: exit 0.
- Example extracted from the guide with a script, copied as `templates/teaching-lab.yml`
  into a scratch repo (schema/, taxonomy.yml, templates/) under
  `.exchange/tmp/task-005/repo`: `validate --root` exit 0, no issue.
- Extra: `uv run pytest` 201 passed, `uv run ruff check .` clean.

Rules checked against the code: name 3-60, description 20-200, tags max 4 unique, `url`
https with no whitespace and no credentials (validate.py), `path` relative without `..`,
`ref` not starting with `-`, `language` `^[a-z]{2}$`, other fields refused
(`additionalProperties: false`), id regex from validate.py, `path` makes detection read the
tree at that subfolder (detect.py `_resolve_tree`), so root of the template means root of
its `path`.

### Artifacts produced

- `/home/omni/specfirst/.worktrees/gallery-contributing/CONTRIBUTING.md` (branch
  `task/005-contributing`, commit `263ab1a`).

### Opinion / concerns

- The six-hour rebuild and the vote refresh are stated as the spec says; no publish
  workflow exists on this branch (TASK-004), so I could not verify them. Only
  `.github/workflows/validate.yml` is present.
- The spec says the id comes from the file name; validate.py enforces the regex on the file
  stem, which matches. The guide's "file name without `.yml`" is consistent.
- `language` is not in the contract's list of fields beyond "optional fields", so I
  included it in the table since the schema has it.
- The URL check in the schema only requires `https://` and no whitespace; "reachable" is
  not verified by validation, only by the scheduled fetch. The guide says so indirectly
  (page falls back to entry fields).

### CLARIFICATION NEEDED

None.
