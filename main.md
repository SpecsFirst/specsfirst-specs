---
project: specsfirst
status: active           # draft | active | completed | archived
version: 0.1.0
owner: aduvermy
created: 2026-11-06
updated: 2026-11-09
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
| `template-in-repo-feature-specs` | `SpecsFirst/template-in-repo-feature-specs` (`../template-in-repo-feature-specs`) | Template: specs inside the code repo, one folder per feature | `subrepos/template-in-repo-feature-specs/main.md` |
| `template-multi-repo-specs` | `SpecsFirst/template-multi-repo-specs` (`../template-multi-repo-specs`) | Template: one specs repo driving several code repos (clean dev-studio specs-template) | `subrepos/template-multi-repo-specs/main.md` |

## Decisions

- 2026-11-06: `gallery` and `.github` are under the MIT license (operator's choice).
- 2026-11-06: this specs repo is under CC0-1.0, like the template: it is mostly text
  that people may copy freely (operator left the choice to the orchestrator).
- 2026-11-06: `template-research-thesis` is under CC0-1.0, with `LICENSE` excluded
  from what Copier copies (operator's choice: people fill a template in to make
  their own specs, so they should owe no notice, and generated projects must not
  inherit this license unasked).

- 2026-11-09: redesign of the gallery (operator brief and mockups, kept in
  `subrepos/gallery/design/`): cards with full file trees, stars as 👍 on Discussions
  without giscus, submission through an issue form. See ADR-006 to ADR-008.

- 2026-11-09: placement and agnostic templates (operator brief). An entry says where its
  specs live (`placement: standalone | in-repo`, optional, default standalone) and may give
  its install `command`. Templates must be model- and provider-agnostic: the submission
  refuses tool-specific files (ADR-009). The bot still never opens a pull request (ADR-008
  kept, operator's choice). Two new templates in the org, under CC0-1.0:
  `template-in-repo-feature-specs` and `template-multi-repo-specs`. The operator writes
  and aligns the gallery entries; tasks never edit `templates/`.

## Referenced specs

- `subrepos/gallery/main.md` — the gallery
- `subrepos/gallery/design/` — the gallery's mockups and logo
- `subrepos/.github/main.md` — the organization profile
- `architecture.md` — decisions and data contracts
