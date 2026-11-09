---
subrepo: template-multi-repo-specs
parent: specsfirst
status: active           # draft | active | completed | archived
version: 0.1.0
owner: aduvermy
created: 2026-11-09
updated: 2026-11-09
tags: [template, software, multi-repo]
---

# template-multi-repo-specs — sub-repo spec

## Purpose / role in the ecosystem

A template listed by the gallery with `placement: standalone`: one specs repo driving several
code repos, with task files as contracts between orchestrator and coder. It is the clean,
licensed, Copier-installable version of `dev-studio/specs-template` (gitlab.inria.fr, read at
`c58cc2d`), which this specs repo itself follows.

## Goal

A team starts a specs repo with one `copier copy` and gets the original's logic unchanged: a
global `main.md`, one folder per code repo under `subrepos/`, task files as exchange
contracts, conventions, environment and audits.

## Scope

- **In scope:** the original's files, cleaned: context-specific references (team names,
  internal URLs, local paths, internal tools) become `<...>` placeholders; anything tied to
  a model or provider is reworded neutrally; `_example_subrepo/` becomes
  `subrepos/{{ first_subrepo }}/`.
- **Out of scope:** the agent roles and tooling that act on these specs (they stay outside
  the template), and any file for one agent tool.

## Interfaces with other sub-repos

Listed by the gallery (entry written by the operator): name "Multi-repo specs", placement
standalone, kind software, tag multi-repo, CC0-1.0.

## Decisions

- 2026-11-09: CC0-1.0, `LICENSE` and `README.md` excluded from what Copier copies (same
  convention as `template-research-thesis`).
- 2026-11-09: `_templates_suffix: ""` like `template-research-thesis`. The sub-repo folder
  is named by the `first_subrepo` variable (operator's brief), so the gallery tree shows
  `{{ first_subrepo }}/` literally.
- 2026-11-09: the answers file is a plain `.copier-answers.yml` rendered from
  `_copier_answers`, so `copier update` works.

- 2026-11-09 (TASK-001): `_exclude` anchors `/LICENSE` and `/README.md`: Copier matches
  gitignore-style, so a bare `README.md` also dropped the task boards.
- 2026-11-09 (TASK-001): the title goes into the front matter as `{{ project_title | tojson }}`,
  valid YAML for any title.

## Code location

- **Remote:** git@github.com:SpecsFirst/template-multi-repo-specs.git (not created yet)
- **Local path:** `specfirst/template-multi-repo-specs` (sibling of the specs repo)
- **Default branch:** main

Tasks in `tasks/` target that repo.

## Dev commands

```bash
uv run --with copier copier copy --defaults --vcs-ref HEAD \
  --data project_title="Test" --data first_subrepo=api . <empty dir>
```

## Referenced specs

- `tasks/README.md` — task board
- `../gallery/main.md` — the gallery that lists this template
- `../../main.md` — the global (ecosystem) spec
