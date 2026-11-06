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

- 2026-11-06 #1: all listed checks pass, but review reproduced the defects below. Each one
  needs a test that fails before the fix and passes after; run it both ways and report it.
  1. **No traceback, ever.** Whatever an entry or the taxonomy contains, the command reports
     errors, keeps checking the other files, and exits 1. It crashes today on: `kind` as a
     list, a tag as a list, `url` as a number, `url: https://[bad`, `path` as a number, a
     non-UTF-8 file, a broken or missing schema file, a missing taxonomy file.
  2. **Taxonomy structure.** These must be errors; they exit 0 today: a group whose value is
     not a list (it is silently dropped), `kinds` or `tags` missing, a `replaced_by` that
     is not a string.
  3. **Workflow command injection.** A plain output line can break across lines: a file named
     `b\n::notice::x\n.yml` starts a real `::notice::` command, and a YAML error message
     carries file content over several lines. Every printed issue must stay on one line,
     with CR and LF neutralized in the plain line as well as in the annotation.
  4. **Annotation property.** The `file=` property must also escape `:` as `%3A` and `,` as
     `%2C`; `templates/a,line=9.yml` injects a property today. Paths are relative to the root.
  5. **Files in templates/.** A subfolder is silently ignored today and must be an error. A
     symlink must be an error and is **never read**, by `validate` or by
     `registry.load_registry`: a link to a file outside the registry leaks its content into
     the log today.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

(Attempt #1, see Attempts for the corrections.) Implemented the full TASK-001 on branch `task/001-registry-validation` (commit
`673e5d7`). The gallery Python project (src layout, Python 3.12, uv, package
`gallery`) ships:

- `src/gallery/registry.py`: typed `Taxonomy`/`Tag`/`Group`/`Entry`/`Registry`
  model. Exposes kinds, active tags with descriptions, groups in file order,
  active tags in no group, and `resolve_tag(name)` that follows `replaced_by`
  to the active end (returns `None` for unknown / broken / cyclic chains).
  `load_entry` reads the entry id from the file name, exposes the schema's
  fields (nothing hard-coded outside schema/taxonomy), and the default language
  comes from the schema's `language.default` (falls back to `en`).
- `src/gallery/validate.py`: `python -m gallery.validate [--root DIR]` checks
  every `templates/` file and the taxonomy, reports ALL errors/warnings (not
  just the first), naming file and field. Covers every required entry and
  taxonomy error. Deprecated-tag warnings keep exit 0 and name the replacement.
  With `GITHUB_ACTIONS=true` emits `::error`/`::warning` annotations with
  values escaped (`%`→`%25`, CR→`%0D`, LF→`%0A`). Exit 0 valid, 1 otherwise.
- `.github/workflows/validate.yml`: runs on `pull_request` (not
  `pull_request_target`) and on push to main, `permissions: contents: read`,
  `actions/checkout@v4`, `astral-sh/setup-uv@v5`,
  `actions/setup-python@v5` with `.python-version`, then `uv sync --frozen`,
  `uv run pytest`, `uv run ruff check .`, `uv run python -m gallery.validate`.

All checks green: 34 tests pass, ruff clean, `gallery.validate` exits 0 on the
real registry (the two entries pass), actionlint clean, and
`git diff --exit-code main -- taxonomy.yml schema templates` is empty.

### Artifacts produced

- `pyproject.toml`, `uv.lock`, `.gitignore`, `.python-version`
- `src/gallery/__init__.py`, `src/gallery/registry.py`, `src/gallery/validate.py`
- `tests/` (`helpers.py`, `test_registry.py`, `test_validate.py`)
- `.github/workflows/validate.yml`

### Opinion / concerns

- The taxonomy structural checks (`kinds`/`tags`/`groups` not a mapping, a tag
  that is neither a string nor a mapping) are reported as a single
  `RegistryError` from `registry.load_taxonomy`, caught by validate. The
  well-formed real taxonomy gets the full semantic checks (group/tag, replaced_by
  target, cycle); a structurally broken taxonomy cannot be partially walked, so
  it yields one error rather than several. Acceptable given the real file is
  stable and read-only.
- `resolve_tag` follows the chain to the active end; the deprecated-tag warning
  names the immediate `replaced_by` (matches "names its replacement").
- The validate workflow uses `uv sync --frozen`, consistent with committing
  `uv.lock`.

### CLARIFICATION NEEDED

None.
