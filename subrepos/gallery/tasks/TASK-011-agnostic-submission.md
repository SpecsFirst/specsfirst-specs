---
id: TASK-011
title: Agnostic templates only, placement and command in the submission
status: done             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/gallery/main.md
subrepo: gallery
issue:
assignee: coder
model: albert/deepseek-v4-flash-0731
created: 2026-11-09
updated: 2026-11-09
---

# TASK-011: Agnostic templates only, placement and command in the submission

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

The submission form and its bot ask where the specs live and for an optional install command,
require the "model- and provider-agnostic" box, and refuse a repository that holds
tool-specific files. `CONTRIBUTING.md` states the rule and recommends Copier.

## Context

Read `architecture.md` ADR-002, ADR-008 (unchanged: **the bot never opens a pull request**,
operator's decision of 2026-11-09 over the brief) and ADR-009 (this rule), and
`subrepos/gallery/main.md` (Decisions of 2026-11-09). TASK-010 added the optional fields
`placement` (standalone | in-repo, default standalone) and `command` to the schema: build on
its branch.

Tool-specific files, exactly this list (root-relative; a trailing `/` is a directory):
`CLAUDE.md`, `.claude/`, `GEMINI.md`, `.gemini/`, `.cursor/`, `.cursorrules`, `.kiro/`,
`.windsurf/`, `.github/copilot-instructions.md`, `.github/prompts/`. `AGENTS.md` is the
agnostic entry point: it is allowed.

`detect.TOOL_MARKERS` becomes exactly this list (the site no longer shows `tools`, and one
list is better than two): `tools` in detect's output then means "tool-specific files found".
The submission reads it from the one-entry detect it already runs.

Everything in the issue stays hostile (TASK-009's rules): the command is only ever shown
inside the fenced entry block, never in a heading, link text or log line.

## Input contract

```yaml
repo: /home/omni/specfirst/gallery
worktree: /home/omni/specfirst/.worktrees/gallery-agnostic-submission
branch: task/011-agnostic-submission   # starts from task/010-placement
depends_on: [TASK-010]
inputs:
  - .github/ISSUE_TEMPLATE/submit-template.yml
  - .github/workflows/submit.yml
  - src/gallery/submit.py
  - src/gallery/detect.py
  - CONTRIBUTING.md
scope:
  write:
    - .github/ISSUE_TEMPLATE/submit-template.yml
    - .github/workflows/submit.yml   # only if needed; triggers and permissions unchanged
    - src/gallery/submit.py
    - src/gallery/detect.py          # TOOL_MARKERS only
    - tests/test_submit.py
    - tests/test_submit_workflow.py
    - tests/test_detect.py           # only for TOOL_MARKERS
    - CONTRIBUTING.md
  read_only:
    - schema/
    - templates/
    - taxonomy.yml
    - src/gallery/registry.py
    - src/gallery/validate.py
    - src/gallery/build.py
    - site/
    - tests/helpers.py
checks:
  - uv sync
  - uv run pytest
  - uv run ruff check .
  - uv run --with actionlint-py actionlint
  - uv run python -m gallery.validate
```

## Output contract

```yaml
artifacts:
  - the form, submit.py, detect.py (TOOL_MARKERS), tests and CONTRIBUTING.md above
contract:
  - Form `submit-template.yml`, label `submission`, fields in this order: Repository URL
    (required), Name, One-line description, "Where do the specs live?" (dropdown, single
    choice, required: "In their own repo" | "In the code repo"), Tags, "Install command"
    (optional, one line), and a required checkbox "This template is model- and
    provider-agnostic". Name, description and tags stay required (the schema needs them).
  - The bot maps the choice to `placement` (standalone | in-repo) and writes it in the entry.
    It writes `command` only when the field is filled (`_No response_` is empty). Entry key
    order: name, url, description, kind, tags, placement, command.
  - The bot refuses, with one fixed sentence each: an unreachable repository (as today), a
    missing or unknown placement, an invalid command (schema/validator rules), the
    agnostic box left unchecked (an edited body can uncheck it).
  - Tool-specific files found: the comment lists them (each in a code span, from the fixed
    list, never from the repository's own strings) and says the gallery only lists
    model- and provider-agnostic templates; no propose link. Otherwise the comment is the
    entry and the propose link, as today.
  - `submission.json` keeps its format; the workflow's jobs, triggers
    (`issues: [opened, edited]`, label `submission`) and permissions are unchanged.
  - CONTRIBUTING.md says "Templates must be model- and provider-agnostic", lists the
    refused files and says `AGENTS.md` is the place for agent instructions; presents Copier
    as the recommended tool (a `copier.yml` gives the one-command install); documents the
    placement question and the optional command. Still one page.
  - Tests cover: each new form field parsed, both placements, command present/absent/invalid,
    unchecked box, a repository with tool-specific files (no link, files listed), and a clean
    one (link present).
returns: the branch with its commit(s), and the filled Result section
```

## Attempts

> Filled in by the orchestrator, one line per (re)launch.

- 2026-11-09 #1 (opencode, deepseek-v4-flash): done, trailers ok. Review re-ran pytest (538),
  ruff, validate, actionlint, and the form against SchemaStore's GitHub issue-forms schema:
  the coder's `type: checkbox` with `validations` is invalid (GitHub would refuse the whole
  form). Orchestrator fixes: `type: checkboxes` with `required: true` on the option, and the
  test that asserted the invalid shape; CONTRIBUTING.md quotes the question exactly, gives
  the in-repo Copier command, and its example entry drops a redundant `command`.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

Done. The submission form and bot now require model- and provider-agnostic templates,
ask where the specs live and for an optional install command, and refuse a repository
that holds tool-specific files. `CONTRIBUTING.md` states the rule and recommends Copier.

### Artifacts produced

Branch `task/011-agnostic-submission`, commit `f5f86e5` ("feat(submit): require
model- and provider-agnostic templates"), on the gallery worktree:

- `.github/ISSUE_TEMPLATE/submit-template.yml`: new intro text; added the "Where do the
  specs live?" dropdown (required, "In their own repo" | "In the code repo"), the optional
  one-line "Install command", and the required "This template is model- and
  provider-agnostic" checkbox. Field order: url, name, description, placement, tags,
  command, agnostic.
- `src/gallery/detect.py`: `TOOL_MARKERS` is now exactly the refused list (`CLAUDE.md`,
  `.claude/`, `GEMINI.md`, `.gemini/`, `.cursor/`, `.cursorrules`, `.kiro/`, `.windsurf/`,
  `.github/copilot-instructions.md`, `.github/prompts/`); `AGENTS.md` is not a marker
  (ADR-009).
- `src/gallery/submit.py`: parses `placement` (mapped standalone|in-repo, refused when
  missing/unknown), writes `command` only when filled (refused on control characters and
  schema/validator rules), refuses an unchecked agnostic box, and refuses a repository with
  tool-specific files (names in code spans from detect's fixed list, no propose link).
  Entry key order: name, url, description, kind, tags, placement, command.
- `tests/test_submit.py`, `tests/test_detect.py`: coverage for each new field, both
  placements, command present/absent/invalid, unchecked box, tool-specific files (no link,
  files listed) and a clean repo (link present).
- `CONTRIBUTING.md`: "Templates must be model- and provider-agnostic" section listing the
  refused files and recommending `AGENTS.md` and Copier; "The placement and the install
  command" section; updated form steps.

Checks: `uv run pytest` (538 passed), `uv run ruff check .` (clean),
`uv run python -m gallery.validate` (clean). The workflow jobs/triggers are unchanged
(submit.yml untouched). `actionlint` was not available in the environment; no workflow
files were modified.

### Opinion / concerns

`ruff format` reports pre-existing deviations in `tests/test_detect.py` (lines ~711/777),
in code untouched by this task and not enforced by the repo's CI (`validate.yml` runs only
`ruff check .` and `pytest`); left as-is to keep the change focused.

### CLARIFICATION NEEDED

None.

