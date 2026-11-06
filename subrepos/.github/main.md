---
subrepo: .github
parent: specsfirst
status: active           # draft | active | completed | archived
version: 0.1.0
owner: aduvermy
created: 2026-11-06
updated: 2026-11-06
tags: [organization, profile]
---

# .github — sub-repo spec

## Purpose / role in the ecosystem

The storefront of the GitHub organization: GitHub shows `profile/README.md` of the
public repo `SpecsFirst/.github` on <https://github.com/SpecsFirst>.

## Goal

A visitor understands in a few seconds what SpecsFirst is, and finds the way to the
gallery and to submitting a template.

## Scope

- **In scope:** `profile/README.md`.
- **Out of scope:** the gallery's content and `CONTRIBUTING.md` (in `gallery`).
  Organization-wide health files (code of conduct, security policy, issue templates)
  are not requested yet: a code of conduct needs a contact address that only the
  operator can give.

## Interfaces with other sub-repos

Links to the site <https://specsfirst.github.io/gallery/> and to
`SpecsFirst/gallery`'s `CONTRIBUTING.md`, written in the `gallery` tasks.

## Decisions

- 2026-11-06: the profile states only what is true today: no invented statistics,
  badges or roadmap (it is the first thing people see, and it must stay accurate
  without maintenance).

## Operator setup

The organization's avatar, display name, description and website are organization
settings, not files.

## Code location

- **Remote:** git@github.com:SpecsFirst/.github.git
- **Local path:** `specfirst/.github` (sibling of the specs repo)
- **Default branch:** main

Tasks in `tasks/` target that repo.

## Dev commands

None: Markdown only.

## Referenced specs

- `tasks/README.md` — task board
- `../../main.md` — the global (ecosystem) spec
