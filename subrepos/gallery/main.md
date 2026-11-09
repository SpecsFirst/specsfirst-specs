---
subrepo: gallery
parent: specsfirst
status: active           # draft | active | completed | archived
version: 0.1.0
owner: aduvermy
created: 2026-11-06
updated: 2026-11-09
tags: [gallery, github-pages, discussions]
---

# gallery — sub-repo spec

## Purpose / role in the ecosystem

The public gallery of spec templates for agent-driven projects: the registry of
entries, the Actions and the static site.

## Goal

People compare templates by their file structure, star the ones they like, and start
one with a single command. Submitting must stay trivial: one issue form, and the gallery
detects everything else itself.

Principles (operator, 2026-11-09): simple and minimal, every element shown helps choose a
template; 100 % static and 100 % GitHub (no backend, no third-party service, no
tracking); JavaScript only for the tag filter and the copy button, every page readable
without it; accessible (real `<a>` and `<button>`, contrast ≥ 4.5:1, targets ≥ 40 px,
correct on mobile).

## Scope

**In scope:**

- Submission through a GitHub issue form (`submit-template.yml`, label `submission`):
  an Action checks the repository and answers on the issue with the entry and a link
  that proposes `templates/<id>.yml` from the author's fork; the author opens the pull
  request. The bot never writes to the repository (ADR-008). A pull request adding the
  file by hand stays possible. The validation
  Action checks every entry against the schema, and `kind` and `tags` against
  `taxonomy.yml`.
- A scheduled Action clones every template with git (any forge) and detects:
  license, `copier.yml`, last update and file tree. If the clone fails, the page
  still shows the entry's own fields.
- Stars: a star is a 👍 (`THUMBS_UP`) reaction on the first post of the template's own
  Discussion in `SpecsFirst/gallery`, created by the publish workflow after the merge.
  Not the stars of the template's repository: templates may live on GitLab. One GitHub
  account counts once; no anonymous vote. Counts are read at build time.
- Home page: header, intro, one tag filter (client side), and a grid of cards sorted by
  stars, then by latest update. Each card shows its metadata and its full file tree.
- Template page: the same card, alone, plus the `copier copy <url> my-project` command
  when a `copier.yml` exists.
- The look is set by the mockups in `design/` (see `design/README.md`).

**Deliverables:** the Actions, the site, and a one-page `CONTRIBUTING.md`.

**Out of scope:** a backend, accounts other than GitHub's, comments on the site,
hosting templates.

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

Project-wide decisions are in `../../architecture.md` (ADR-001 to ADR-008).

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
- 2026-11-06: a publish fails, and the live site stays as it is, when no template at all
  could be fetched or when the votes cannot be read. Both point at a broken runner or API,
  and publishing would blank every page's data or the ranking. One template failing alone
  still publishes, with that page showing the entry's own fields.

- 2026-11-09: redesign (operator brief and mockups in `design/`). The `main.md`
  preview, the "Detected information" block (tools, language, commit), giscus and its
  comments leave the site. Detect still produces `tools` and `preview` (data contract
  unchanged); the site ignores them.
- 2026-11-09: the site shows `kind` as the first tag of a template, and the filter offers
  kinds and tags alike: on the site a kind is a tag. Entries keep both fields (the schema
  is the operator's and unchanged).
- 2026-11-09: the filter offers only the kinds and tags used by at least one listed
  template, in `taxonomy.yml` order (kinds, then tags). One tag at a time. No URL state,
  no sort control: the order is fixed at build time.
- 2026-11-09: the file tree is rendered like `LC_ALL=C tree` from detect's `tree` list,
  and hides dot-entries (files and folders whose name starts with `.`), like `tree`
  without `-a`. The summary line counts only what is shown. A truncated listing says so
  under the summary.
- 2026-11-09: the Copier command uses `my-project` instead of `<name>`, so it can be
  pasted as is.

- 2026-11-09 (TASK-009): `validate.entry_issues()` returns the issues of one entry, each with
  optional `field` and `code`, so the submission comment uses fixed sentences and never
  quotes the validator's messages (they contain user values). A closed issue produces no
  comment.

## Design tokens

Light values are the operator's; dark values are the orchestrator's, each pair checked
at ≥ 4.5:1. Every color of the site is one of these tokens (CSS custom properties).

| Token | Light | Dark | Use |
|-------|-------|------|-----|
| `--bg` | `#F7F6F2` | `#151412` | page, tree block |
| `--card` | `#FFFFFF` | `#1F1E1B` | cards, star button |
| `--border` | `#E7E4DC` | `#34322D` | card and button borders (radius 12 px) |
| `--text` | `#1C1B19` | `#EDEBE4` | text |
| `--muted` | `#6B6860` | `#A8A49A` | secondary text |
| `--accent` | `#3B4FD8` | `#8FA2FF` | logo, Submit button, links, active filter |
| `--accent-text` | `#FFFFFF` | `#151412` | text on the accent |
| `--warn` | `#B45309` | `#F0A04B` | license "None" |
| `--star-fill` | `#F5C542` | `#F5C542` | star, count > 0 |
| `--star-stroke` | `#B7791F` | `#E0A030` | star outline |
| `--star-bg` | `#FFFBEB` | `#2B2414` | star button, count > 0 |
| `--star-border` | `#F3D98B` | `#6B5417` | star button border, count > 0 |
| `--term-bg` | `#1C1B19` | `#0E0D0C` | command block |
| `--term-text` | `#EDEBE4` | `#EDEBE4` | command text |
| `--term-prompt` | `#8FA2FF` | `#8FA2FF` | the `$` |

Tag dots use a fixed palette of six: `#0F766E`, `#7C3AED`, `#B45309`, `#2F7D4F`,
`#3B4FD8`, `#BE185D`. A tag's color is `palette[i % 6]`, where `i` is its position in
`taxonomy.yml` (kinds in file order, then tags in file order, deprecated ones included).
It is stable as long as new kinds and tags are added at the end of their list. The dots
are decorative: the tag name is always written next to them.

Fonts: Inter (text) and JetBrains Mono (tree, command), served by the site itself.

## Operator setup

Repository settings that agents do not change:

1. Pages: source "GitHub Actions". Done 2026-11-06.
2. Discussions: enable, and create a category for templates (Announcement type, so
   only giscus and maintainers can create threads). Done 2026-11-06.
3. Install the [giscus app](https://github.com/apps/giscus) on `SpecsFirst/gallery`.
   Done 2026-11-06. No longer needed once TASK-007 is merged: it may be uninstalled.
4. Pull requests: merge commits only, head branches deleted. Done 2026-11-06, on all
   four repos.
5. Rulesets on `main`: Organization admin on the bypass list, "For pull requests only".
   Done 2026-11-06, on all four repos. See `../../conventions.md` § Protected branches.
6. Label `submission` on `SpecsFirst/gallery` (the issue form applies it only if it
   exists; it only routes issues to `submit.yml`). Done 2026-11-09.
7. Keep "Allow GitHub Actions to create and approve pull requests" off (organization
   policy, 2026-11-09): nothing in the gallery needs it. Settings → Actions → General →
   "Approval for running fork pull request workflows from contributors": "Require
   approval for first-time contributors". Done 2026-11-09.
8. Check that `github-actions[bot]` can create a Discussion in the Templates category
   (Announcement type: only maintainers create threads there). The first publish after
   TASK-008 shows it; if it fails, change the category type to "Open-ended discussion"
   or "General". For TASK-008.

## Discussions configuration

Public values, read from the GitHub API on 2026-11-06 (in `site.yml`, under `giscus:`
until TASK-007 renames the key to `discussions:`):

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
- `design/README.md` — mockups and logo
- `../../main.md` — the global (ecosystem) spec
- `../../architecture.md` — decisions and data contracts
