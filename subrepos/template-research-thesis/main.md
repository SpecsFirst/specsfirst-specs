---
subrepo: template-research-thesis
parent: specsfirst
status: active           # draft | active | completed | archived
version: 0.1.0
owner: aduvermy
created: 2026-11-06
updated: 2026-11-06
tags: [template, research, thesis]
---

# template-research-thesis — sub-repo spec

## Purpose / role in the ecosystem

The first template listed by the gallery (entry `templates/research-thesis.yml`): a
specs repo kept as a lab notebook, with pre-registered hypotheses, bench protocols,
and tasks for humans and agents.

## Goal

A researcher starts a thesis specs repo with one `copier copy` and understands the
method by reading the files: a hypothesis is written down before the work, each piece
of work is a task done by a human or an agent, and results update the hypothesis.

## Scope

- **In scope:** a generic skeleton, generalized from a real thesis structure given by
  the operator (a chemoresistance study). It works for any experimental thesis, with
  bench or field work.
- **Out of scope:** discipline-specific content, code (it lives in the sibling code
  repo the specs describe), and files for one specific agent tool: the template is
  model-agnostic, so no `CLAUDE.md` or `.opencode/`.

## Interfaces with other sub-repos

The gallery reads this repo as any other template. It expects `main.md`, `copier.yml`
and a license file **at the repo root**, which rules out Copier's `_subdirectory`.

## Decisions

- 2026-11-06: CC0-1.0, with `LICENSE` excluded from what Copier copies (see
  `../../main.md`).
- 2026-11-06: `_templates_suffix: ""` so that files keep their real names: the gallery
  finds `main.md` and shows a readable preview. A `main.md.jinja` would be invisible
  to it.
- 2026-11-06: directory names are fixed (`analysis-code/`, not `{{ code_repo }}/`) so
  that the file tree shown by the gallery stays readable. People rename them freely.

## Code location

- **Remote:** git@github.com:SpecsFirst/template-research-thesis.git
- **Local path:** `specfirst/template-research-thesis` (sibling of the specs repo)
- **Default branch:** main

Tasks in `tasks/` target that repo.

## Dev commands

```bash
uv run --with copier copier copy --defaults --vcs-ref HEAD \
  --data project_title="Test" --data researcher="Test" . <empty dir>
```

## Referenced specs

- `tasks/README.md` — task board
- `../gallery/main.md` — the gallery that lists this template
- `../../main.md` — the global (ecosystem) spec
