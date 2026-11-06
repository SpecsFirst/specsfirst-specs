---
id: TASK-001
title: Registry and validation Action
status: open             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/gallery/main.md
subrepo: gallery
issue:
assignee: coder
model: albert/deepseek-v4-flash-0731
created: 2026-11-06
updated: 2026-11-06
---

# TASK-001: Registry and validation Action

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

Create the Python project of the gallery, with a module loading the registry and a
validation command run by a GitHub Action on every pull request.

## Context

Read `subrepos/gallery/main.md` (scope, decisions) and, in `architecture.md`,
ADR-001 (stack) and ADR-005 (deprecated tags). Later tasks build on the registry
module for detection and the site, so it must be a clean library and not only a
script.

`taxonomy.yml`, `schema/template.schema.json` and `templates/*.yml` already exist
and **must not be modified**. A tag in `taxonomy.yml` is either `name: description`
or, when deprecated, the mapping form of ADR-005.

## Input contract

```yaml
repo: /home/omni/specfirst/gallery
worktree: /home/omni/specfirst/.worktrees/gallery-registry-validation
branch: task/001-registry-validation
depends_on: []
inputs:
  - taxonomy.yml
  - schema/template.schema.json
  - templates/*.yml
scope:
  write:
    - pyproject.toml
    - uv.lock
    - .gitignore
    - .python-version
    - src/gallery/
    - tests/
    - .github/workflows/validate.yml
  read_only:
    - taxonomy.yml
    - schema/
    - templates/
checks:
  - uv run pytest
  - uv run ruff check .
  - uv run python -m gallery.validate
  - uv run --with actionlint-py actionlint .github/workflows/validate.yml
  - git diff --exit-code main -- taxonomy.yml schema templates
```

## Output contract

```yaml
artifacts:
  - pyproject.toml, uv.lock, .gitignore  # Python 3.12, src layout, package `gallery`
  - src/gallery/registry.py
  - src/gallery/validate.py
  - tests/
  - .github/workflows/validate.yml
contract:
  - registry.py loads taxonomy and entries into typed objects, with functions the
    other modules can call, and takes the repo root as a parameter (tests use
    tmp_path, never the real registry files for failure cases).
  - registry.py exposes for taxonomy: kinds; active tags with descriptions; groups
    in file order; `resolve_tag(name)` following replaced_by to the end of the chain.
    Active tags in no group are exposed too, so that no tag vanishes from filters.
  - registry.py exposes for an entry: id, the schema's fields, and the default
    language `en`. No field name, kind or tag is hard-coded outside the schema and
    taxonomy files.
  - `python -m gallery.validate [--root DIR]` checks every file of templates/ and
    the taxonomy, reports ALL errors (not only the first), naming file and field, and
    exits 0 if valid, 1 otherwise.
  - Entry errors: invalid YAML; not a mapping; fails the JSON Schema (draft 2020-12,
    using the file schema/template.schema.json, not a copy); id not matching
    ^[a-z0-9]+(-[a-z0-9]+)*$; extension other than .yml in templates/; kind not in
    taxonomy; tag not in taxonomy; url containing credentials; path absolute or
    containing `..`; ref starting with `-`.
  - A deprecated tag on an entry is a warning, exit code stays 0, and the message
    names its replacement.
  - Taxonomy errors: invalid structure; a group naming an unknown tag; a deprecated
    tag without replaced_by; replaced_by naming an unknown tag; a replaced_by cycle.
  - When running inside GitHub Actions (env GITHUB_ACTIONS=true), errors and warnings
    are also printed as ::error file=...:: and ::warning file=...:: annotations.
    Values read from files are sanitized so that an entry cannot inject workflow
    commands (escape %, CR, LF as GitHub documents).
  - validate.yml runs on `pull_request` (never `pull_request_target`) and on push to
    main, with `permissions: contents: read`, using astral-sh/setup-uv and pinned
    action versions; it runs pytest, ruff, then the validation.
  - Tests cover each error above, the deprecated-tag warning, resolve_tag, and the two
    real entries passing.
returns: the branch with its commits, and the filled Result section
```

## Steps (suggested)

1. Project skeleton with uv (`jsonschema`, `pyyaml`; dev: `pytest`, `ruff`).
2. `registry.py`, with tests.
3. `validate.py`, with tests.
4. `validate.yml`.

## Attempts

> Filled in by the orchestrator, one line per (re)launch.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

### Artifacts produced

### Opinion / concerns

### CLARIFICATION NEEDED
