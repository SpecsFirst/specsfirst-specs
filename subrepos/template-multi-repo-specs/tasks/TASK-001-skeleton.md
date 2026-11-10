---
id: TASK-001
title: Clean multi-repo specs skeleton
status: done             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/template-multi-repo-specs/main.md
subrepo: template-multi-repo-specs
issue:
assignee: coder
model: anthropic/claude-sonnet-5-5
created: 2026-11-09
updated: 2026-11-09
---

# TASK-001: Clean multi-repo specs skeleton

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

Turn the original `dev-studio/specs-template` into a clean, Copier-installable template:
same logic, sections and rules; no context-specific or tool-specific reference.

## Context

Read `subrepos/template-multi-repo-specs/main.md`.

The original, at commit `c58cc2d`, is copied (without `.git`) in
`subrepos/template-multi-repo-specs/tasks/.exchange/original/` of the specs repo. **Start
from its real content.**

`/home/omni/specfirst/template-research-thesis` is the reference for tone, README structure,
`copier.yml` and the Jinja convention (`_templates_suffix: ""`): reproduce it exactly.
`main.md` is the entry point, like there.

Target tree:

```
.
├── .copier-answers.yml          # content: {{ _copier_answers|to_nice_yaml -}}
├── .gitignore
├── LICENSE
├── README.md
├── architecture.md
├── audit
│   └── AUDIT-template.md
├── conventions.md
├── copier.yml
├── environment.md
├── main.md
└── subrepos
    └── {{ first_subrepo }}
        ├── main.md
        └── tasks
            ├── README.md
            └── TASK-000-template.md
```

Cleaning rules (operator's brief):

- Keep the original's sections and rules. Remove references tied to one context (team
  names, internal URLs, local paths, the internal agent kit and its scripts by name) and
  replace them with `<...>` placeholders. Examples: `agents-kit` → `<agent tooling>`;
  `agents-kit/scripts/agent-trailers trailers` → "your agent tooling"; "symlinked onto the
  VM" → gone; "no opencode config" → "no file for one agent tool".
- Nothing may depend on one model, provider or agent tool; reword neutrally. Forge names
  (GitLab, GitHub) are not providers in this sense: keep the forge-specific merge request
  instructions, tightened.
- Harmonize with research-thesis: `main.md` as entry point (it says what to read next),
  same placeholder conventions, concise tone. The original `README.md` is about the
  template: its useful parts (concept, layout, workflow, what is versioned) move into the
  generated files (mostly `main.md` and the boards); the new README is about the template.
- `_example_subrepo` becomes `subrepos/{{ first_subrepo }}/`; the global `main.md` sub-repos
  table and the sub-repo `main.md` use `{{ first_subrepo }}` where the original had
  `<name>`. `{{ project_title }}` replaces the project name placeholders in `main.md`.
- `architecture.md` points to a `decisions/` folder that does not exist: fix it.

`copier.yml`: `project_title` (required, no default); `first_subrepo` (required, no
default, validated with the regex `^[a-z0-9][a-z0-9-]*$` and a clear message);
`_templates_suffix: ""`; `_exclude` with Copier's defaults (as research-thesis) plus
`LICENSE` and `README.md`.

`README.md`: the purpose in 3 lines; the tree; install:
`copier copy https://github.com/SpecsFirst/template-multi-repo-specs my-project`; update:
`copier update`; how to add a sub-repo (copy `subrepos/<first_subrepo>/`, rename it, then
list it in `main.md`); then "Develop the template" and "License" like research-thesis.

## Input contract

```yaml
repo: /home/omni/specfirst/template-multi-repo-specs
worktree: /home/omni/specfirst/.worktrees/template-multi-repo-specs-skeleton
branch: task/001-skeleton
depends_on: []
inputs:
  - subrepos/template-multi-repo-specs/main.md
  - subrepos/template-multi-repo-specs/tasks/.exchange/original/ (read only)
  - /home/omni/specfirst/template-research-thesis (read only)
scope:
  write:
    - .copier-answers.yml
    - .gitignore
    - README.md
    - architecture.md
    - audit/
    - conventions.md
    - copier.yml
    - environment.md
    - main.md
    - subrepos/
  read_only:
    - LICENSE                    # CC0-1.0, committed by the orchestrator
checks:
  - `uv run --with copier copier copy --defaults --vcs-ref HEAD --data project_title="Test
    project" --data first_subrepo=api <worktree> <empty scratch dir>` exits 0
  - the generated tree is the target tree with `subrepos/api/`, plus `.copier-answers.yml`;
    no copier.yml, README.md or LICENSE
  - `--data first_subrepo="Bad Name"` is refused by the validator
  - `grep -rE '\{\{|\}\}|\{%'` in the generated files finds nothing
  - `git init`, commit, then `uv run --with copier copier update --defaults` exits 0
  - `grep -riE 'claude|gemini|cursor|copilot|kiro|windsurf|openai|anthropic|opencode|sonnet|deepseek|albert|inria|dev-studio|agents-kit|aduvermy|/home/'`
    on the template finds nothing (LICENSE aside)
```

## Output contract

```yaml
artifacts:
  - the files of the target tree, except LICENSE
contract:
  - In English, Markdown. Skeleton files stay short; conventions.md keeps every rule of the
    original but no repetition (aim for 100 lines or fewer).
  - Every section of each original file still exists (or its content moved, said in Result).
  - Placeholders `<...>`; Copier variables only `{{ project_title }}` and
    `{{ first_subrepo }}`. No other `{{`, `{%` or `{#` anywhere (every file is rendered).
  - The tree in README.md matches the real files.
  - Result lists, file by file, what was removed or reworded compared with the original.
returns: the branch with its commit(s), and the filled Result section
```

## Attempts

> Filled in by the orchestrator, one line per (re)launch.

- 2026-11-09 #1 (Claude Sonnet subagent, parallel to gallery TASK-010 and in-repo TASK-001):
  done, two commits, trailers stamped by the orchestrator. Review re-ran copy, update after
  commit (source given as an absolute path: a relative `.` is stored as is in the answers
  file), validator, Jinja and vendor greps: all pass. Orchestrator fix: the title is
  rendered with `| tojson` in both front matters, so any title stays valid YAML (checked
  with `Thesis: part #2 "draft"`). Accepted: anchored `/LICENSE` and `/README.md`, and
  conventions.md at 116 lines.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

Branch `task/001-skeleton` of `/home/omni/specfirst/template-multi-repo-specs`
(worktree `/home/omni/specfirst/.worktrees/template-multi-repo-specs-skeleton`), 2 commits:
`b22b53e` feat: add multi-repo specs skeleton; `86e1838` fix: anchor template-only excludes
to the repo root.

Checks (all run on `86e1838`):

- `copier copy --defaults --vcs-ref HEAD --data project_title="Test project" --data
  first_subrepo=api` exits 0. Generated: `.copier-answers.yml`, `.gitignore`,
  `architecture.md`, `audit/AUDIT-template.md`, `conventions.md`, `environment.md`,
  `main.md`, `subrepos/api/{main.md,tasks/README.md,tasks/TASK-000-template.md}`; no root
  `copier.yml`, `README.md` or `LICENSE`.
- `first_subrepo="Bad Name"` refused by the validator (message: "Use only lowercase
  letters, digits and hyphens, ..."), exit 1, nothing generated.
- `grep -rE '\{\{|\}\}|\{%|\{#'` on the generated files: no match.
- `git init`, commit, `copier update --defaults`: exit 0.
- Forbidden-term grep on the template (LICENSE aside): no match.
- Deviation caught by the checks: the spec's `_exclude: README.md` also drops
  `subrepos/*/tasks/README.md` (gitignore-style, matches at any depth). `copier.yml` uses
  the anchored `/LICENSE` and `/README.md`.
- `conventions.md` is 116 lines (aim was 100 or fewer): every rule kept, the Merge
  requests section alone is 40 lines. See "Opinion".

What changed compared with the original, file by file:

- `.copier-answers.yml` (new): `{{ _copier_answers|to_nice_yaml -}}`.
- `copier.yml` (new): as brief; research-thesis excludes plus `/LICENSE`, `/README.md`.
  `project_title` and `first_subrepo` required, no default; regex validator on the latter.
- `.gitignore`: same patterns; header "specs-template .gitignore" and the comment block
  rewritten (no template name), duplicated comment lines merged.
- `README.md`: rewritten, now about the template (purpose, install, `copier update`, tree
  of what you get, add a sub-repo, develop, license). Removed: "Relationship to agents-kit",
  the paragraph on the single `AGENTS.md` symlinked onto the VM, "no `CLAUDE.md`, no
  opencode config" (now "nothing specific to one agent tool"), the link to `../agents-kit`.
  Moved to `main.md`: the concept ("ecosystem of sub-repos, one specs repo"), the layout
  tree (with the Layout table's purposes as tree comments, table dropped), Workflow (4
  steps) and Versioning/gitignore (now "What is versioned"; `environment.md` added to the
  list, it was missing). "Copy this template to `<project>-specs/`" became the Copier
  install.
- `main.md`: `project: {{ project_title }}`, title `{{ project_title }} — global spec`;
  intro now says what to read next (entry point); prose prompts in `<...>` in backticks
  (thesis convention); sub-repos table row and tree use `{{ first_subrepo }}`; added
  "How this repo works" (concept, tree, Workflow, What is versioned) and a dated
  "Decisions" section (as research-thesis). "Referenced specs" now also lists
  `conventions.md`/`environment.md`; the "Fill one row per sub-repo" note became a one-line
  sync reminder.
- `architecture.md`: the `decisions/` pointer removed ("Record each significant choice
  here as an ADR"); prompts in `<...>` backticks; nothing else.
- `conventions.md` (every section kept; trimmed and reworded):
  - Naming: the trailers' fenced example block dropped (the table now holds each trailer's
    format); `<exact model id> via <provider> <address>` became `<agent identity>
    <address>`; "stamped by the launcher, never typed by the model" became "stamped by
    `<agent tooling>`, never typed by the agent"; `<agents-kit URL>` became `<agent tooling
    URL>`; `; kit dirty` became `; tooling dirty`; the sentence "older commits keep that
    form, still accepted by `check`" removed (tied to the internal script), as was the
    "Task ... `<specs repo>/-/blob/...`" duplicate wording; commit-style and spec-commit
    bullets kept.
  - Audits: `agents-kit/scripts/agent-trailers trailers` became `<agent tooling>`;
    "the model" became "the agent".
  - Process: unchanged.
  - Merge requests: "The agent opens the MR (GitHub: PR)"; GitLab and GitHub opening
    instructions split into two tightened bullets (same facts); example deploy command
    `make deploy HOST=<alias> TAGS=<roles>` became "a deploy command".
  - Definitions: unchanged.
  - "Model-agnostic note" became "Tool-agnostic note" (points to `<agent tooling>`).
- `environment.md`: "(provider quota)" became a `<reason: quota, shared machine>`
  placeholder; "no `/tmp`" became "no system temp dir"; "Heavy envs and model caches"
  became "Heavy environments and caches" (package manager's cache, uv as example);
  recipe example made generic.
- `audit/AUDIT-template.md`: front-matter `Co-authored-by: <agent identity>` and
  `Assisted-by: <agent tooling URL>/-/commit/<sha> (<tool> <version>)`; the rest verbatim
  (verbatim apart from the two placeholders).
- `subrepos/{{ first_subrepo }}/main.md` (was `_example_subrepo/main.md`): `subrepo:` and
  `parent:` use `{{ first_subrepo }}` / `{{ project_title }}`; title `{{ first_subrepo }} —
  sub-repo spec`; the parent pointer comment fixed to `../../main.md`; local path
  `<project>/{{ first_subrepo }}`; prompts in `<...>` backticks; "Referenced specs" gains
  `tasks/README.md`.
- `subrepos/{{ first_subrepo }}/tasks/README.md`: content unchanged apart from em dashes
  replaced by colons, "Depends on" `—` became `none`, `<title>` in backticks, and a pointer
  to `TASK-000-template.md`.
- `subrepos/{{ first_subrepo }}/tasks/TASK-000-template.md`: byte-identical to the original
  (nothing tool- or context-specific in it).

### Artifacts produced

- Branch `task/001-skeleton` in `/home/omni/specfirst/template-multi-repo-specs`: all files
  of the target tree except `LICENSE` (already there).
- Scratch: `tasks/.exchange/tmp/out` (generated test project).

### Opinion / concerns

- `project: {{ project_title }}` and `parent: {{ project_title }}` are plain YAML scalars:
  a title containing `: ` or `#` would break the front matter. Quoting them would be
  safer but the title itself could contain a quote; the brief allows only the bare
  variable, so left as is.
- `conventions.md` is 116 lines, not 100: the trailers table and the GitLab/GitHub merge
  request instructions carry real rules; cutting more would drop rules.
- The Task trailer URL keeps GitLab's `/-/blob/` form in the table (original); the
  paragraph below gives the GitHub variant.
- The `{{ first_subrepo }}` folder name is a real directory with braces in the git tree;
  fine on Linux/macOS, may be awkward on Windows checkouts.

### CLARIFICATION NEEDED

None.
