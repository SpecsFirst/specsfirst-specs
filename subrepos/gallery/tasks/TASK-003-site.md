---
id: TASK-003
title: Static site
status: open             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/gallery/main.md
subrepo: gallery
issue:
assignee: coder
model: anthropic/claude-sonnet-5-5
created: 2026-11-06
updated: 2026-11-06
---

# TASK-003: Static site

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

Add `python -m gallery.build`, which turns the registry, `detected.json` and `votes.json`
into the static site: the gallery page and one page per template.

## Context

Read `subrepos/gallery/main.md` (scope, decisions, giscus configuration) and, in
`architecture.md`, ADR-001 (Jinja2, plain JS), ADR-002 (templates are hostile), ADR-004
(giscus mapping), ADR-005 (deprecated tags) and § Data / interfaces.

Other tasks are running on other branches of this repo. TASK-001 corrects `registry.py` and
`validate.py`, and TASK-002 adds `detect.py`, which is absent from your base. **Do not
modify or depend on those changes.** Use `load_registry`, `Taxonomy` and `entry.fields` as
they are. A real `detected.json` is provided as input; `votes.json` does not exist yet, so
build your own sample following the documented format.

No browser is available: test the HTML output in Python. The orchestrator will test the
JavaScript and the rendering in a browser at review, so keep the JavaScript small and plain.

## Input contract

```yaml
repo: /home/omni/specfirst/gallery
worktree: /home/omni/specfirst/.worktrees/gallery-site
branch: task/003-site
depends_on: [TASK-001]
inputs:
  - src/gallery/registry.py
  - tests/helpers.py
  - /home/omni/specfirst/specsfirst-specs/subrepos/gallery/tasks/.exchange/tmp/task-003/detected.json
scope:
  write:
    - pyproject.toml
    - uv.lock
    - .gitignore
    - site.yml
    - site/
    - src/gallery/build.py
    - tests/test_build.py
  read_only:
    - src/gallery/registry.py
    - src/gallery/validate.py
    - tests/helpers.py
    - tests/test_registry.py
    - tests/test_validate.py
    - taxonomy.yml
    - schema/
    - templates/
checks:
  - uv sync
  - uv run pytest
  - uv run ruff check .
  - uv run python -m gallery.build --detected <the provided detected.json> --output <scratch dir>
```

Dependencies to add: `jinja2`, `markdown-it-py`. Nothing else, and no CDN or web font.

## Output contract

```yaml
artifacts:
  - src/gallery/build.py
  - site/                # Jinja2 templates, site.css, site.js
  - site.yml             # public site configuration
  - tests/test_build.py
contract:
  - `python -m gallery.build [--root DIR] [--detected FILE] [--votes FILE] --output DIR`
    writes `index.html`, `templates/<id>/index.html`, `assets/site.css`, `assets/site.js`,
    `.nojekyll`, and a byte-identical copy of `schema/template.schema.json` at
    `schema/template.schema.json` (the schema's $id points there).
  - A missing file, or an id missing from a file, means "no data", never an error. The
    site builds with neither file: entries show their own fields, with 0 votes.
  - The site is served under /gallery/: every internal link is relative, so that it works
    under any prefix. No <base> element.
  - site.yml holds the giscus configuration of main.md and the site's title and tagline. The
    build reads it: these values are not in the code or templates.

  # Gallery page
  - One card per template: name, description, kind, tags, votes, license, last update, a link
    to its page. Copier support and detected tools appear when present.
  - Filters come only from taxonomy.yml: kinds first, then the tags by group in file order,
    then active tags in no group under "Other". Descriptions from the taxonomy are visible as
    labels' title or help text. A test proves it by building with a different taxonomy, and no
    kind, tag or group name appears in code, templates or JS.
  - Deprecated tags are never offered as filters. On a card and a page, a deprecated tag shows
    as its `resolve_tag` target, with duplicates removed. A card's filter data uses the same
    resolved names, so that an old entry is found by the new tag.
  - Filtering: one kind at a time or all; several tags select templates holding ALL of them. A
    live count says how many are shown, announced through aria-live. When nothing matches, a
    message says so and offers to clear the filters.
  - Sort: by votes (default) or by latest update, both descending. Ties and missing values
    come last, ordered by name. The server-rendered order is the votes order.
  - Filters and sort live in the URL query, so that a view can be shared and survives reload.
  - Without JavaScript, the full ranked list is readable and every link works; controls that
    cannot work are hidden.

  # Template page
  - Shows: name, description, kind, tags, language, a link to the repo (at `ref` when set, as
    text, since forges differ), the preview, the detected info (license, Copier, tools, last
    update, short commit, file tree, with a note when truncated), and the giscus thread.
  - `path` and `ref` are shown when set.
  - Copier command, only when `copier` is true: `copier copy <url> <name>` by default, with
    `<name>` literally. With `ref`: `copier copy --vcs-ref <ref> <url> <name>`. With `path`:
    `git clone <url> <name>-src && copier copy <name>-src/<path> <name>` (with
    `git -C <name>-src checkout <ref>` as a middle step when `ref` is also set). In a <code>
    block, with a copy button that JavaScript adds.
  - When status is "failed", or there is no data: a short neutral notice and the entry's own
    fields. No error code from a stranger's repo is displayed raw except the closed list, in
    plain words. The giscus thread is still there.
  - giscus: the official script from https://giscus.app/client.js with `data-mapping="specific"`,
    `data-term="<id>"`, `data-strict="1"`, `data-reactions-enabled="1"`, `data-input-position="top"`,
    `data-theme="preferred_color_scheme"`, `data-loading="lazy"`, and the values from site.yml.
    The page says that voting uses 👍 on the thread and needs a GitHub account.

  # Security (ADR-002): every value below comes from strangers
  - Jinja2 autoescape is on for all templates. Nothing from an entry or from detected.json
    passes through `|safe`, except the preview HTML from the sanitizing renderer below.
  - Preview: markdown-it-py, raw HTML disabled, with tables enabled and YAML front matter removed
    before rendering. Links keep only absolute https/http and mailto targets, with
    rel="nofollow noopener noreferrer"; any other target, including a relative one, renders as
    its plain text. Images render as their alt text: no remote request and no tracking pixel.
  - The preview is marked with the entry's `language`, in a lang attribute.
  - The repo link only uses a `url` that starts with https://.
  - Every page has a Content-Security-Policy meta tag allowing only: its own assets, and
    giscus.app for scripts, frames and styles. base-uri and object-src are `none`, and
    inline scripts are not allowed: the JS is in assets/site.js.
  - No inline event handler. The JS uses textContent or classes, never innerHTML with data.

  # Quality
  - Responsive from 320 px with no horizontal page scroll; long values wrap, and wide content
    (tree, command, tables) scrolls inside its own block. Light and dark through
    prefers-color-scheme. Visible keyboard focus, real labels on controls, sufficient contrast,
    semantic headings and landmarks. A restrained, readable design that looks made on purpose,
    with system fonts.
  - The gallery page links to CONTRIBUTING.md on GitHub as the way to submit a template.

  # Tests
  - Using fixture registries in tmp_path, they cover: both pages' content; the two "no data"
    cases and a failed template; a deprecated tag on a card; filters from a different
    taxonomy; each form of the Copier command, and none when copier is false; the giscus attributes;
    the byte-identical schema; relative links only; the CSP tag on every page; and hostile values:
    `<script>` and `<img onerror>` in name, description, file tree and preview, a `javascript:`
    link, a `data:` link, a relative link and a remote image in the preview. They parse the HTML
    (html.parser) rather than only searching for strings.
returns: the branch with its commit(s), and the filled Result section
```

## Attempts

> Filled in by the orchestrator, one line per (re)launch.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

### Artifacts produced

### Opinion / concerns

### CLARIFICATION NEEDED
