---
project: specsfirst
status: active           # draft | active | completed | archived
version: 0.1.0
owner: aduvermy
created: 2026-11-06
updated: 2026-11-06
tags: [specs, templates, gallery, agents]
---

# SpecsFirst — global spec

> The **global, whole-project** view. A project is an **ecosystem of
> sub-repos**; this file summarizes the entire project and points to each
> sub-repo's own spec.

## Goal

Make spec templates for agent-driven projects easy to share and find, through a
public gallery where submitting stays trivial.

## Scope

- **In scope:** the gallery (submission, detection, votes, site), and the
  organization's public profile.
- **Out of scope:** hosting the listed templates (any forge), and a backend (the
  site is static, on GitHub Pages).

## Context

GitHub org [SpecsFirst](https://github.com/SpecsFirst), everything public. The
gallery is served at <https://specsfirst.github.io/gallery/>.

## Sub-repos

| Sub-repo | Code repo | Role in ecosystem | Spec |
|----------|-----------|-------------------|------|
| `gallery` | `SpecsFirst/gallery` (`../gallery`) | Registry of template entries, Actions and static site | `subrepos/gallery/main.md` |
| `.github` | `SpecsFirst/.github` (`../.github`) | Organization storefront: the public profile page | `subrepos/.github/main.md` |
| `template-research-thesis` | `SpecsFirst/template-research-thesis` (`../template-research-thesis`) | First template of the gallery: a specs repo kept as a lab notebook | `subrepos/template-research-thesis/main.md` |

## Open points (operator)

- `https://github.com/SpecsFirst/template-research-thesis`, listed by the entry
  `research-thesis.yml`, does not exist yet and must be created. Until it does,
  this entry stays in the "clone failed" state and shows only its own fields.

## Decisions

- 2026-11-06: `gallery` and `.github` are under the MIT license (operator's choice).
- 2026-11-06: this specs repo is under CC0-1.0, like the template: it is mostly text
  that people may copy freely (operator left the choice to the orchestrator).
- 2026-11-06: `template-research-thesis` is under CC0-1.0, with `LICENSE` excluded
  from what Copier copies (operator's choice: people fill a template in to make
  their own specs, so they should owe no notice, and generated projects must not
  inherit this license unasked).

## Referenced specs

- `subrepos/gallery/main.md` — the gallery
- `subrepos/.github/main.md` — the organization profile
- `architecture.md` — decisions and data contracts
