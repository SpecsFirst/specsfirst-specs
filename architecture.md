# Architecture

> Architecture decisions and structural overview. Keep this aligned with
> `main.md`.

## Overview

The gallery has no backend. A GitHub Actions workflow turns the registry into a
static site, and giscus (hosted by giscus.app, stored in GitHub Discussions)
provides votes and comments.

```
pull request ──> validate.yml ──> taxonomy.yml + schema + templates/*.yml

schedule / push on main ──> publish.yml
   registry (templates/*.yml, taxonomy.yml)
     ├── detect ──> detected.json   (git clone of each template, any forge)
     ├── votes  ──> votes.json      (GraphQL: 👍 on each Discussion)
     └── build  ──> _site/          ──> GitHub Pages artifact
```

## Decisions

### ADR-001: Python, uv and a generated static site

- **Status:** accepted
- **Date:** 2026-11-06
- **Context:** validation, detection and page generation are small scripts. The
  agents' VM has Python and uv, but no Node.
- **Decision:** Python 3.12 managed by uv. The site is generated with Jinja2;
  filters and sorting are plain JavaScript, with no framework and no JS build
  step.
- **Consequences:** one language and no `node_modules`. The page must work
  without a bundler, and its JS logic stays small.

### ADR-002: Templates are untrusted: never check out, never execute

- **Status:** accepted
- **Date:** 2026-11-06
- **Context:** the scheduled workflow fetches repositories chosen by strangers
  and publishes what it reads from them. A `main.md` symlink pointing at a runner
  file would publish that file if read from a checkout, and a preview can carry
  HTML or scripts.
- **Decision:** detection reads git objects only, in a bare repository
  (`git ls-tree`, `git cat-file`): no checkout, no submodules, no LFS, no hooks,
  and no code from the template runs. A symlink is listed but never followed.
  Every fetch has a timeout and size limits, with prompts disabled. Rendered
  previews have raw HTML disabled and unsafe link schemes refused, and everything
  else from a template is escaped. The fetch steps get no write token.
- **Consequences:** a very large or slow repository is reported as failed, and
  the page shows the entry's own fields.

### ADR-003: Publish through a Pages artifact, with no bot commits

- **Status:** accepted
- **Date:** 2026-11-06
- **Context:** detected data changes on every run; committing it would fill the
  history and need a write token next to untrusted data.
- **Decision:** one workflow runs detection, votes and build, then deploys with
  `actions/deploy-pages`. Generated data is not versioned.
- **Consequences:** Pages must use the source "GitHub Actions". A template whose
  clone fails loses its detected info until the next successful run. GitHub
  disables schedules after 60 days without repository activity; re-enabling is an
  operator action.

### ADR-004: One Discussion per template, found by its id

- **Status:** accepted
- **Date:** 2026-11-06
- **Context:** the 👍 count must rank the gallery, and giscus creates a Discussion
  on the first reaction or comment.
- **Decision:** giscus uses `data-mapping="specific"`, `data-term="<id>"` and
  `data-strict="1"`, in one dedicated Discussions category of `SpecsFirst/gallery`.
  The vote count is the `THUMBS_UP` reaction count of the Discussion whose title
  is exactly `<id>`, read with GraphQL by `publish.yml`. No Discussion means zero
  votes.
- **Consequences:** ranking refreshes on each publish, not live; the giscus thread
  itself is live. The ids of the repo and category are public configuration,
  stored in the repo, available only once the operator has enabled Discussions.

### ADR-005: Form of a deprecated tag

- **Status:** accepted
- **Date:** 2026-11-06
- **Context:** `taxonomy.yml` writes a tag as `name: description` and says tags are
  deprecated with `replaced_by`, without giving the form.
- **Decision:** a deprecated tag becomes a mapping, and both forms are valid:

  ```yaml
  tags:
    thesis: A PhD thesis — one person, several years, hypotheses.
    phd:
      description: Old name of thesis.
      deprecated: true
      replaced_by: thesis
  ```

  An entry using a deprecated tag still passes validation, with a warning. Unknown
  tags fail. The site shows the end of the `replaced_by` chain. Filters only offer
  active tags.
- **Consequences:** old entries never break. `replaced_by` chains without a target
  or with a cycle fail validation.

## Repository / module structure

```
gallery/
├── taxonomy.yml, schema/template.schema.json, templates/<id>.yml
├── src/gallery/           # registry, validate, detect, votes, build
├── site/                  # Jinja2 templates, CSS, JS
├── tests/
├── .github/workflows/     # validate.yml, publish.yml
└── CONTRIBUTING.md
```

## Data / interfaces

`id` is the entry's file name without `.yml`.

`detected.json`, written by detect, read by build:

```json
{
  "generated_at": "2026-11-06T12:00:00Z",
  "templates": {
    "<id>": {
      "status": "ok",
      "error": null,
      "license": "MIT",
      "copier": true,
      "tools": ["AGENTS.md", "CLAUDE.md"],
      "last_update": "2026-10-01T09:30:00Z",
      "commit": "<full sha>",
      "tree": ["main.md", "subrepos/"],
      "tree_truncated": false,
      "preview": "<raw main.md or null>"
    }
  }
}
```

With `status: "failed"`, `error` holds a short reason and the other detected
fields are `null`. A `license` that is present but not identified is `"unknown"`;
no license file is `null`.

`votes.json`, written by votes, read by build:

```json
{
  "generated_at": "2026-11-06T12:00:00Z",
  "templates": { "<id>": { "votes": 3, "discussion_url": "https://github.com/..." } }
}
```

The build treats a missing file or a missing id as "no data", never as an error.
