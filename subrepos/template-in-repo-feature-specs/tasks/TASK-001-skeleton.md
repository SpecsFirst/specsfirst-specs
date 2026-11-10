---
id: TASK-001
title: In-repo feature specs skeleton
status: done             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/template-in-repo-feature-specs/main.md
subrepo: template-in-repo-feature-specs
issue:
assignee: coder
model: anthropic/claude-sonnet-5-5
created: 2026-11-09
updated: 2026-11-09
---

# TASK-001: In-repo feature specs skeleton

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

Write the template's files and its `copier.yml`, so that `copier copy <url> .` at the root of
an existing code repo adds `AGENTS.md` and a `specs/` folder, and `copier update` works.

## Context

Read `subrepos/template-in-repo-feature-specs/main.md` (decisions matter here).

`/home/omni/specfirst/template-research-thesis` is the reference for tone, README structure,
`copier.yml` and the Jinja rendering convention (`_templates_suffix: ""`): reproduce its
convention exactly. Its files are short skeletons with `<...>` placeholders, wrapped in
backticks in body text so they stay visible when rendered.

Target tree (the answers file is the only addition):

```
.
├── .copier-answers.yml          # content: {{ _copier_answers|to_nice_yaml -}}
├── AGENTS.md
├── LICENSE
├── README.md
├── copier.yml
└── specs
    ├── conventions.md
    ├── decisions
    │   └── ADR-001-short-name.md
    ├── features
    │   └── 001-example-feature
    │       ├── design.md
    │       ├── requirements.md
    │       └── tasks.md
    └── product.md
```

Content, from the operator's brief:

- **AGENTS.md**, under 30 lines: read `specs/product.md` then `specs/conventions.md` first;
  for a feature, read requirements → design → tasks, in that order; update task statuses
  while working; never edit `requirements.md` without human approval; record any structural
  decision in an ADR.
- **specs/product.md**: title `{{ project_title }}`; Goal (1–2 sentences); Users; Scope
  (in / out); feature index table: ID, name, status, link.
- **specs/conventions.md**: numbering `NNN-short-name`; feature statuses `draft`,
  `approved`, `in-progress`, `done`; task statuses `todo`, `doing`, `done`, `blocked`; who
  changes what (humans own requirements; agents propose the design and update tasks); when to
  write an ADR; how to create a feature (copy `001-example-feature` and rename it).
- **requirements.md**: status and date; Problem; user stories; numbered, testable
  acceptance criteria (R1…); Out of scope.
- **design.md**: Approach; components or files touched; data and interfaces; Risks;
  Alternatives considered; references to requirement IDs.
- **tasks.md**: a checklist; each task has an ID (T1…), a status, the requirements it covers
  and a done criterion.
- **ADR-001-short-name.md**: Status, Date, Context, Decision, Consequences.
- **README.md**: the purpose in 3 lines; the tree; install from the root of the code repo:
  `copier copy https://github.com/SpecsFirst/template-in-repo-feature-specs .`; update:
  `copier update`; if an `AGENTS.md` already exists, Copier asks before overwriting it: answer
  no and merge by hand. Then "Develop the template" and "License" like research-thesis.
- **copier.yml**: `project_title` (required: no default); `_templates_suffix: ""`;
  `_exclude` with Copier's defaults (as research-thesis) plus `LICENSE` and `README.md`.

## Input contract

```yaml
repo: /home/omni/specfirst/template-in-repo-feature-specs
worktree: /home/omni/specfirst/.worktrees/template-in-repo-feature-specs-skeleton
branch: task/001-skeleton
depends_on: []
inputs:
  - subrepos/template-in-repo-feature-specs/main.md
  - /home/omni/specfirst/template-research-thesis (read only: copier.yml, README.md, main.md)
scope:
  write:
    - .copier-answers.yml
    - AGENTS.md
    - README.md
    - copier.yml
    - specs/
  read_only:
    - LICENSE                    # CC0-1.0, committed by the orchestrator
checks:
  - in a scratch dir, `git init` a fake code repo with `src/main.py`, `README.md` and one
    commit, then `uv run --with copier copier copy --defaults --vcs-ref HEAD
    --data project_title="Test product" <worktree> <scratch dir>`
  - the scratch dir gains exactly AGENTS.md, .copier-answers.yml and specs/ (tree below);
    its own src/ and README.md are untouched; no copier.yml or LICENSE copied
  - `grep -rE '\{\{|\}\}|\{%'` in the generated files finds nothing;
    specs/product.md starts with `# Test product`
  - commit in the scratch repo, then `uv run --with copier copier update --defaults` exits 0
  - `grep -riE 'claude|gemini|cursor|copilot|kiro|windsurf|openai|anthropic|opencode'`
    on the template finds nothing (LICENSE aside)
```

## Output contract

```yaml
artifacts:
  - the files of the target tree, except LICENSE
contract:
  - In English, Markdown, short skeletons: no file over 40 lines except README.md;
    AGENTS.md under 30 lines.
  - Placeholders `<...>` (backticked in body text); the only Copier variable is
    `{{ project_title }}`, used in specs/product.md (and nowhere it would break `copier update`).
  - Neutral towards models and providers; no file for one agent tool.
  - The tree in README.md matches the real files (dotfiles aside, as `tree` shows them).
  - Statuses and file names used in AGENTS.md, conventions.md and the example feature agree.
returns: the branch with its commit(s), and the filled Result section
```

## Attempts

> Filled in by the orchestrator, one line per (re)launch.

- 2026-11-09 #1 (Claude Sonnet subagent, parallel to gallery TASK-010 and multi-repo TASK-001):
  done, one commit, trailers stamped by the orchestrator. Review re-ran copy into a fake code
  repo, update after commit, Jinja and vendor greps: all pass. Awaiting operator review before
  the repo is created on GitHub.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

Done. Branch `task/001-skeleton`, commit `75bd38f` (`feat: add in-repo feature specs skeleton`).
All checks passed in a scratch repo: copy adds exactly `AGENTS.md`, `.copier-answers.yml` and
`specs/` (the fake repo's `src/` and `README.md` untouched; no `copier.yml`/`LICENSE` copied); no
`{{`, `}}` or `{%` left; `specs/product.md` starts with `# Test product`; after committing,
`copier update --defaults` exits 0; the vendor-word grep finds nothing. `AGENTS.md` is 24 lines,
no other file but `README.md` exceeds 40 lines.

### Artifacts produced

`.copier-answers.yml`, `AGENTS.md`, `README.md`, `copier.yml`, `specs/product.md`,
`specs/conventions.md`, `specs/decisions/ADR-001-short-name.md`,
`specs/features/001-example-feature/{requirements,design,tasks}.md` in the worktree.

### Opinion / concerns

- `copier update` printed "No git tags found in template; using HEAD": expected until the
  template repo gets a version tag (e.g. `v0.1.0`); worth tagging before publishing.
- `.copier-answers.yml` is a bare filename, so with `copier update` the answers file name is
  fixed by Copier's default; fine for a single template per repo.
- The scratch check was first run with a mistyped path (a failed `cd`); it only produced
  read-only output and nothing was modified. Reran correctly.

### CLARIFICATION NEEDED

None.
