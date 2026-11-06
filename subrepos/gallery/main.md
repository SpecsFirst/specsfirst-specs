---
subrepo: gallery
parent: specsfirst
status: active           # draft | active | completed | archived
version: 0.1.0
owner: aduvermy
created: 2026-11-06
updated: 2026-11-06
tags: [gallery, github-pages, giscus]
---

# gallery — sub-repo spec

## Purpose / role in the ecosystem

The public gallery of spec templates for agent-driven projects: the registry of
entries, the Actions and the static site.

## Goal

People share templates, filter them, and vote. Submitting must stay trivial: an
entry is five lines, and the gallery detects everything else itself.

## Scope

**In scope:**

- Submission by pull request: one file `templates/<id>.yml`. A validation Action
  checks it against the schema, and checks `kind` and `tags` against
  `taxonomy.yml`.
- A scheduled Action clones every template with git (any forge) and detects:
  license, `copier.yml`, tool-specific files (`CLAUDE.md`, `AGENTS.md`,
  `.opencode/`…), last update, file tree and `main.md` preview. If the clone
  fails, the page still shows the entry's own fields.
- Votes and comments with giscus (one Discussion per template). The 👍 count
  ranks the gallery.
- Filters built from `taxonomy.yml` (kinds, then tags by group), never hard-coded.
  Sort by votes or by latest update.
- Template page: preview, tags, detected info, the giscus thread, and the
  `copier copy <url> <name>` command when a `copier.yml` exists.

**Deliverables:** the Actions, the site, and a one-page `CONTRIBUTING.md`.

**Out of scope:** a backend, accounts other than GitHub's (giscus), hosting
templates.

## Evolution rules

- New fields are always optional.
- Tags are deprecated (with `replaced_by`), never deleted or renamed.
- Display groups live in `taxonomy.yml`, not in the entries.
- Anything detectable is detected, not asked.
- The registry is centralized: a breaking change, if ever needed, is one migration
  pull request over all entries.

## Interfaces with other sub-repos

`.github` links to this repo's site and `CONTRIBUTING.md`. Data contracts between
the steps are in `../../architecture.md` § Data / interfaces.

## Decisions

`taxonomy.yml`, `schema/template.schema.json` and the two initial entries were
given by the operator and committed as they are: tasks never change them.

Project-wide decisions are in `../../architecture.md` (ADR-001 to ADR-005).

- 2026-11-06: an entry's id matches `^[a-z0-9]+(-[a-z0-9]+)*$` (it ends up in URLs
  and Discussion titles).
- 2026-11-06: beyond the schema, validation refuses a `url` with credentials, a
  `path` that is absolute or contains `..`, and a `ref` starting with `-` (these
  values reach git commands).
- 2026-11-06: the site also publishes `schema/template.schema.json` at its own
  path, because the schema's `$id` points there.
- 2026-11-06: an entry with `ref` gets `copier copy --vcs-ref <ref> <url> <name>`.
  Copier cannot target a subfolder of a remote URL, so an entry with `path` gets
  `git clone <url> <name>-src && copier copy <name>-src/<path> <name>` instead.

## Operator setup

Repository settings that agents do not change:

1. Pages: source "GitHub Actions". Done 2026-11-06.
2. Discussions: enable, and create a category for templates (Announcement type, so
   only giscus and maintainers can create threads). Done 2026-11-06.
3. Install the [giscus app](https://github.com/apps/giscus) on `SpecsFirst/gallery`.
   Done 2026-11-06.
4. Pull requests: merge commits only, head branches deleted. Done 2026-11-06, on all
   four repos.

## giscus configuration

Public values, read from the GitHub API on 2026-11-06:

```yaml
repo: SpecsFirst/gallery
repo_id: R_kgDOVlEEaQ
category: Templates
category_id: DIC_kwDOVlEEac4DKhwb
```

## Code location

- **Remote:** git@github.com:SpecsFirst/gallery.git
- **Local path:** `specfirst/gallery` (sibling of the specs repo)
- **Default branch:** main

Tasks in `tasks/` target that repo.

## Dev commands

```bash
uv sync
uv run pytest
uv run ruff check .
uv run python -m gallery.validate
```

## Referenced specs

- `tasks/README.md` — task board
- `../../main.md` — the global (ecosystem) spec
- `../../architecture.md` — decisions and data contracts
