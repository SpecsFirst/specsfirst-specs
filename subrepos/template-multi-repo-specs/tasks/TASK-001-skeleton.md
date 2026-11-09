---
id: TASK-001
title: Clean multi-repo specs skeleton
status: open             # open | in_progress | blocked | done | needs_review | rejected
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

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

### Artifacts produced

### Opinion / concerns

### CLARIFICATION NEEDED
