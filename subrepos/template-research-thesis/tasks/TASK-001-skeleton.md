---
id: TASK-001
title: Generic lab-notebook skeleton
status: done             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/template-research-thesis/main.md
subrepo: template-research-thesis
issue:
assignee: coder
model: anthropic/claude-sonnet-5-5
created: 2026-11-06
updated: 2026-11-06
---

# TASK-001: Generic lab-notebook skeleton

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

Write the template's files and its `copier.yml`, generalizing the real structure below
so that any experimental thesis can use it.

## Context

Read `subrepos/template-research-thesis/main.md` (decisions matter here) and
`/home/omni/specfirst/gallery/templates/research-thesis.yml` (the promise made by the
gallery entry).

The real structure given by the operator, from a chemoresistance thesis. Its shape is
what to keep; its subject is what to remove:

```
resistance-chimio-specs/
├── main.md                                   goal, scope, hypothesis status
├── conventions.md                            append-only, commits, executor
├── environment.md                            storage, cluster, instruments
├── hypotheses/
│   ├── H1-efflux.md                          ✗ refuted
│   ├── H2-metabolism.md                      ✓ supported
│   └── H3-gene-regulator.md                  ✓ supported
├── lab/                                      bench work
│   ├── protocols/
│   │   ├── dose-response.md
│   │   ├── metabolic-assay.md
│   │   └── gene-knockdown.md
│   └── tasks/
│       ├── board.md
│       ├── TASK-003-dose-response-assay.md   👤 human
│       └── …
└── resistance-analysis-code/                  code work
        ├── main.md                           reusable pipelines
        └── tasks/
            ├── board.md
            ├── TASK-004-viability-pipeline.md   🤖 agent
            ├── TASK-005-viability-analysis.md   👤 human
            └── …
```

## Input contract

```yaml
repo: /home/omni/specfirst/template-research-thesis
worktree: /home/omni/specfirst/.worktrees/template-research-thesis-skeleton
branch: task/001-skeleton
depends_on: []
inputs:
  - subrepos/template-research-thesis/main.md
  - /home/omni/specfirst/gallery/templates/research-thesis.yml
scope:
  write:
    - README.md
    - copier.yml
    - main.md
    - conventions.md
    - environment.md
    - hypotheses/
    - lab/
    - analysis-code/
  read_only:
    - LICENSE
checks:
  - uv run --with copier copier copy --defaults --vcs-ref HEAD --data project_title="Test thesis" --data researcher="Test person" . <empty scratch dir>
  - in the generated dir, `grep -rE '\{\{|\}\}'` finds nothing
  - the generated dir has main.md, and has no copier.yml, LICENSE or README.md
```

## Output contract

```yaml
artifacts:
  - copier.yml
  - README.md                 # about the template; not copied into projects
  - main.md                   # goal, scope, table of hypotheses with their status
  - conventions.md            # append-only rule, commits, executor, statuses
  - environment.md            # storage, compute, instruments
  - hypotheses/H1-short-name.md
  - lab/protocols/short-name.md
  - lab/tasks/board.md
  - lab/tasks/TASK-001-bench-example.md          # executor human
  - analysis-code/main.md     # reusable pipelines; the code itself lives in a sibling repo
  - analysis-code/tasks/board.md
  - analysis-code/tasks/TASK-002-pipeline-example.md   # executor agent
  - analysis-code/tasks/TASK-003-analysis-example.md   # executor human
contract:
  - In English. No reference to chemoresistance, cancer, or any specific discipline.
    Examples stay neutral, using angle-bracket placeholders the researcher replaces.
  - No file for a specific agent tool (no CLAUDE.md, AGENTS.md, .opencode/).
  - copier.yml has `_templates_suffix: ""`, the questions project_title and researcher,
    and `_exclude` listing Copier's defaults (copier.yml, copier.yaml, ~*, *.py[co],
    __pycache__, .git, .DS_Store, .svn) plus LICENSE and README.md.
  - main.md at the root renders readably as a preview in spite of its `{{ project_title }}`.
  - A hypothesis file is written before the work. It holds the statement, the prediction,
    what would refute it, the registration date, and a status open | supported | refuted.
    Its registered part is never edited afterwards; results are appended, dated, and link
    to the tasks they come from. This is what "pre-registered" means here.
  - conventions.md states the rule above, and that a refuted hypothesis is kept, never
    deleted.
  - A task names its executor (human or agent) and the hypothesis it serves. An agent task
    has explicit input and output contracts and checks; a human task names its protocol and
    where its raw data goes. The human and agent task formats share the same header fields.
  - lab/ is described as work beyond code, bench or fieldwork, renamable.
  - environment.md and main.md mention where raw data, code and results are deposited, as
    placeholders. They hold no real path, host or credential.
  - The file tree in README.md matches the real files, and README.md shows the
    `copier copy https://github.com/SpecsFirst/template-research-thesis <name>` command.
  - No file over 60 lines except README.md. This is a skeleton, not a manual.
returns: the branch with its commit(s), and the filled Result section
```

## Attempts

> Filled in by the orchestrator, one line per (re)launch.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

Done. Branch `task/001-skeleton`, commit `d16640f` (`feat: add generic lab-notebook skeleton and copier config`) in `/home/omni/specfirst/.worktrees/template-research-thesis-skeleton`.

Checks, run after committing:
- `copier copy --defaults --vcs-ref HEAD --data project_title="Test thesis" --data researcher="Test person" . <scratch>` exits 0.
- `grep -rE '\{\{|\}\}'` on the generated dir finds nothing (main.md renders as `# Test thesis`, `Researcher: Test person`).
- The generated dir has `main.md`, and no `copier.yml`, `LICENSE` or `README.md`.
- No file over 60 lines except README.md (67 lines; the next largest is 57).
- A grep for discipline and agent-tool words (chemo, cancer, drug, cell, dose, claude, opencode, agents.md) finds nothing outside LICENSE (the word "cancellation").

Design choices:
- All three task files share one header: id, title, status, executor, hypothesis, depends_on, created, updated. Bodies differ: human tasks have Protocol and Raw data; the agent task has input/output contracts and checks.
- Task numbers are unique across the whole repo (stated in conventions.md), hence TASK-001 in lab/ and TASK-002/003 in analysis-code/.
- Placeholders are wrapped in backticks in body text so they stay visible when rendered (an unknown `<tag>` would be hidden by Markdown). Frontmatter placeholders are bare angle brackets.
- Task statuses are open | in_progress | blocked | done | rejected (no `needs_review`), a deliberate simplification for a researcher.
- No `.gitignore` or `.exchange/` convention added: outside the write scope.

### Artifacts produced

In the worktree: `copier.yml`, `README.md`, `main.md`, `conventions.md`, `environment.md`, `hypotheses/H1-short-name.md`, `lab/protocols/short-name.md`, `lab/tasks/board.md`, `lab/tasks/TASK-001-bench-example.md`, `analysis-code/main.md`, `analysis-code/tasks/board.md`, `analysis-code/tasks/TASK-002-pipeline-example.md`, `analysis-code/tasks/TASK-003-analysis-example.md`.

### Opinion / concerns

- Copier only substitutes `{{ project_title }}` and `{{ researcher }}`, but with `_templates_suffix: ""` every file is rendered by Jinja. A researcher who later adds `{{`, `{%` or `{#` to a template file would break it; this matters only if the template is extended, not for generated projects.
- The `--vcs-ref HEAD` check copies the committed state, so it must be run after committing (it was).
- `copier.yml` questions have no defaults, so `--defaults` without `--data` fails; that is intended, but worth knowing for the gallery.
- The hypothesis "registered part never edited" is only a convention; git history is the real guard. Conventions.md also says not to rewrite history after registration.

### CLARIFICATION NEEDED

None.
