---
subrepo: template-in-repo-feature-specs
parent: specsfirst
status: active           # draft | active | completed | archived
version: 0.1.0
owner: aduvermy
created: 2026-11-09
updated: 2026-11-09
tags: [template, software, in-repo]
---

# template-in-repo-feature-specs — sub-repo spec

## Purpose / role in the ecosystem

A template listed by the gallery with `placement: in-repo`: specs kept inside the code repo,
one folder per feature with requirements, design and tasks.

## Goal

A team runs `copier copy https://github.com/SpecsFirst/template-in-repo-feature-specs .` at
the root of an existing code repo and gets `AGENTS.md` plus a `specs/` folder that any agent
or human can follow: product first, then conventions, then one feature at a time.

## Scope

- **In scope:** `AGENTS.md`, `specs/` (product, conventions, one example feature, one ADR),
  `copier.yml`, `README.md`, `LICENSE`.
- **Out of scope:** code (`src/`: the template installs into an existing repo), and any
  file for one agent tool or model provider (`CLAUDE.md`, `.claude/`, `GEMINI.md`, …):
  `AGENTS.md` is the only entry point.

## Interfaces with other sub-repos

Listed by the gallery (entry written by the operator): name "In-repo feature specs",
placement in-repo, kind software, CC0-1.0.

## Decisions

- 2026-11-09: CC0-1.0, `LICENSE` and `README.md` excluded from what Copier copies (same
  convention as `template-research-thesis`, and a code repo has its own).
- 2026-11-09: `_templates_suffix: ""` like `template-research-thesis`.
- 2026-11-09: the answers file is a plain `.copier-answers.yml` rendered from
  `_copier_answers`, so `copier update` works and the file stays hidden in the gallery tree.

## Code location

- **Remote:** git@github.com:SpecsFirst/template-in-repo-feature-specs.git (not created yet)
- **Local path:** `specfirst/template-in-repo-feature-specs` (sibling of the specs repo)
- **Default branch:** main

Tasks in `tasks/` target that repo.

## Dev commands

```bash
uv run --with copier copier copy --defaults --vcs-ref HEAD \
  --data project_title="Test" . <code repo dir>
```

## Referenced specs

- `tasks/README.md` — task board
- `../gallery/main.md` — the gallery that lists this template
- `../../main.md` — the global (ecosystem) spec
