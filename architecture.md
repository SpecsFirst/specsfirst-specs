# Architecture

> Architecture decisions and structural overview. Keep this aligned with
> `main.md`.

## Overview

The gallery has no backend and uses no service but GitHub. GitHub Actions turn an
issue form into a pull request, and the registry into a static site. Stars are 👍
reactions on one GitHub Discussion per template.

```
issue (form submit-template.yml, label submission) ──> submit.yml
   check   (contents: read)  ──> submission.json   (validate + detect one entry)
   comment (issues: write)   ──> one bot comment: the entry, or the errors,
                                 and a link that proposes the file from a fork

pull request (from the author's fork) ──> validate.yml ──> taxonomy.yml + schema + templates/*.yml
   merged by the operator only (ruleset on main)

schedule / push on main ──> publish.yml
   discussions (write)  ──> one Discussion per registry id, created if missing
   build (read)
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

- **Status:** superseded by ADR-006 (2026-11-09) for how a Discussion is created; the
  title rule and the vote count stay.
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

### ADR-006: Stars without giscus: the publish workflow creates each Discussion

- **Status:** accepted
- **Date:** 2026-11-09
- **Context:** the redesign drops the comment thread from the site (operator brief). With
  giscus gone, nothing creates a template's Discussion on the first reaction, yet the star
  button must always open one.
- **Decision:** a job of `publish.yml`, before the build, creates the missing Discussions
  (title exactly `<id>`, in the category of ADR-004) with the `GITHUB_TOKEN` and
  `discussions: write`. It reads only the registry of `main` (merged, reviewed), never a
  template. It is idempotent and keeps ADR-004's rule (the oldest Discussion of a title
  counts). The star button is a plain link to the Discussion (`discussion_url` of
  `votes.json`), opened in a new tab. The site loads no script from another origin.
- **Consequences:** a star needs a GitHub account, counted once by GitHub. Counts refresh on
  each publish (push on `main`, and every 6 hours since reactions trigger no workflow). If
  the job fails, the build still runs and a template without a Discussion links to the
  repository's Discussions page.

### ADR-007: No third-party origin at runtime

- **Status:** accepted
- **Date:** 2026-11-09
- **Context:** the operator wants a site that is 100 % static, 100 % GitHub, with no
  tracking. The mockups load Inter and JetBrains Mono from Google Fonts, which would send
  every visitor's IP address to Google.
- **Decision:** the site serves its own WOFF2 fonts (Latin subsets of the weights it uses),
  with their SIL OFL 1.1 license next to them. The Content-Security-Policy allows `'self'`
  only for scripts, styles, fonts and images: no inline script, no inline style attribute.
  Links to github.com and to templates' forges are ordinary links.
- **Consequences:** a few hundred KB of fonts in the repo. Tag colors are classes, not
  `style` attributes.

### ADR-008: Submission by issue form, in two jobs

- **Status:** accepted
- **Date:** 2026-11-09
- **Context:** opening a pull request with a YAML file is a barrier for many authors. An
  issue form is easier, but its text comes from anyone, and the repository to check is
  chosen by them (ADR-002). The organization does not let Actions create pull requests
  (operator, 2026-11-09), and the gallery must not become a place where a bot writes on
  behalf of strangers.
- **Decision:** the bot never writes to the repository: no branch, no commit, no pull
  request. `submit.yml` runs on `issues: [opened, edited]` when the issue has the label
  `submission` (put by the form; it routes, it does not authorize). Job `check` has
  `contents: read` only and no token in the environment of its steps: it parses the issue
  body from the event file (never through `${{ }}` in a script), builds the entry,
  validates it with the registry's validator, and runs detect on that one entry. Job
  `comment` has `issues: write` only, checks out nothing and runs no repository code: it
  posts (or updates) one comment, made by `check`, with the entry and what was detected,
  or the list of problems. When the entry is valid, the comment carries a link to GitHub's
  "new file" page, prefilled with `templates/<id>.yml`: the author clicks it, GitHub forks
  the repository for them, and they open the pull request themselves ("Closes #N").
- **Consequences:** every change to `main` is a pull request from someone's fork, checked
  by `validate.yml` with a read-only token, and merged by the operator (ruleset). A token
  that can only comment on issues is the whole write surface of the submission. The
  author does one more click than with a bot pull request. The id, license, date, Copier
  and tree are still never typed by the author.

## Repository / module structure

```
gallery/
├── taxonomy.yml, schema/template.schema.json, templates/<id>.yml
├── src/gallery/           # registry, validate, detect, votes, discussions, submit,
│                          # tree, build
├── site/                  # Jinja2 templates, CSS, JS, fonts
├── tests/
├── .github/ISSUE_TEMPLATE/submit-template.yml
├── .github/workflows/     # validate.yml, publish.yml, submit.yml
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

`submission.json`, written by `python -m gallery.submit` in the `check` job of
`submit.yml`, read by its `comment` job:

```json
{
  "issue": 12,
  "ok": true,
  "id": "teaching-lab",
  "comment": "<Markdown of the issue comment, propose link included>"
}
```

With `ok: false`, `id` is `null` and `comment` lists every problem found.
