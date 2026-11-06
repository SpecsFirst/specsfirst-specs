---
id: TASK-003
title: Static site
status: done             # open | in_progress | blocked | done | needs_review | rejected
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

- 2026-11-06 #1: done. Browser review in headless Chromium, on the real data and on a hostile
  fixture: 33/33 checks (filters, sort, URL state, no JS, 320 px, hostile values, copy button,
  giscus under the CSP with no violation). The orchestrator fixed layout defects in a separate
  commit: filters pushed the templates far down, doubled title, doubled Repository label, facts
  misaligned on cards. Filters are now collapsed on narrow screens unless a link sets them.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

Done, commit `ee0566c` on branch `task/003-site` (worktree
`/home/omni/specfirst/.worktrees/gallery-site`), one commit on top of `0a47ce1`.
`python -m gallery.build [--root DIR] [--detected FILE] [--votes FILE] --output DIR`
writes `index.html`, `templates/<id>/index.html`, `assets/site.css`, `assets/site.js`,
`.nojekyll` and a byte-identical `schema/template.schema.json`.

Checks actually run, in the worktree:

- `uv sync`, `uv lock --check`: ok. Dependencies added: `jinja2`, `markdown-it-py` only.
- `uv run pytest`: 94 passed (37 of them new, in `tests/test_build.py`, HTML parsed with
  `html.parser`).
- `uv run ruff check .`: all checks passed.
- `uv run python -m gallery.build --detected <provided detected.json> --output
  tasks/.exchange/tmp/task-003/out`: exit 0, 2 template pages. Also run with a sample
  `votes.json` (`out2/`): the voted template moves first.
- Mutation check of the security tests: with autoescape off, raw HTML on, the
  sanitizer removed, images left on, or the `https` check removed on the repo link,
  the test suite fails each time (5, 2, 4, 3 and 1 failures); restored afterwards.
- JavaScript: parsed with esprima (syntax ok), and run in QuickJS against a hand-made
  DOM stub (scratch `tasks/.exchange/tmp/task-003/jssmoke.py`, not committed, not a
  browser): default order, sort by update with missing dates last, kind plus several
  tags (ALL), URL query read on load and written on change, empty message and clear,
  unknown query values ignored, copy button. All asserted and passing.
- Colour contrast computed for every text/background pair in both schemes: text >= 6.3:1,
  chips >= 9:1, accent links >= 7:1; focus ring >= 4.4:1; form-control border >= 4.8:1.

NOT verified (no browser available): real rendering and 320 px layout, the JS in a real
browser, and that giscus works under the CSP (see Opinion).

Design choices to know about:

- `--root` is the registry root. `site/` and `site.yml` are read from `--root` when they
  exist there, else from the repo that holds `gallery.build` (so tests with a tmp
  registry use the real templates and config).
- Links: `templates/<id>/` and `../../` (directory links, no `index.html`), assets
  `assets/...` / `../../assets/...`. No `<base>`.
- Filter state in the URL: `?kind=<k>&tag=<t>&tag=<t>&sort=updated` (`sort=votes` is the
  default and omitted); written with `history.replaceState`.
- Card filter data is `data-kind`, `data-tags` (JSON array of resolved tags), `data-votes`,
  `data-updated` (epoch seconds, empty when unknown), `data-name` (casefolded), `data-id`.
  Order is votes desc, then casefolded name, then id; same comparator in Python and JS.
- Copier command values are shell-quoted (`shlex.quote`), and no command is shown if the
  url is not plain https or the ref starts with `-` or the path is absolute or has `..`.
  Beyond the contract, as defence in depth (entries are strangers' values).
- Entries whose id does not match the id pattern are skipped with a stderr warning (an id
  becomes a directory name). An unparseable data file is a `BuildError` (exit 1); only a
  missing file is "no data".
- Failed templates: the `error` code is mapped to plain words through the closed list
  (`timeout`, `too_large`, `unreachable`, `ref_not_found`, `path_not_found`,
  `invalid_entry`); any other value, and the code itself, is never displayed.
- Preview headings are shifted down two levels (h1 becomes h3) so the page keeps one h1;
  tables sit in a scrolling `div`; `style` attributes of table cells are dropped.
  `markdown-it`'s `validateLink` is bypassed so that a refused link keeps its text
  instead of showing its markdown source.
- Tags unknown to the taxonomy are shown as written and are not filterable.
- The label "Other" for ungrouped tags is in `build.py` (the contract fixes it).
- The two copies of "Submit a template" link to
  `https://github.com/<giscus.repo from site.yml>/blob/main/CONTRIBUTING.md`.

### Artifacts produced

- `src/gallery/build.py`
- `site/base.html`, `site/index.html`, `site/template.html`, `site/assets/site.css`,
  `site/assets/site.js`
- `site.yml`
- `tests/test_build.py`
- `pyproject.toml`, `uv.lock` (two dependencies)
- scratch, not committed: `tasks/.exchange/tmp/task-003/out/`, `out2/`, `votes.json`,
  `jssmoke.py`

### Opinion / concerns

- CSP and giscus: `style-src 'self' https://giscus.app` and no `'unsafe-inline'`, as the
  contract says. If giscus' `client.js` sets inline styles or injects a `<style>`, the
  browser will block that and the thread may look unstyled. This is the first thing to
  check in the browser at review; if it breaks, the fix is one directive in
  `site/base.html`.
- The preview of a repo is only text-styled: image alt text appears inline with no
  marker, so a README that is mostly badges reads as stray words. Acceptable per the
  contract, worth knowing.
- `lstrip_blocks`/`trim_blocks` leave a few odd blank lines in the generated HTML around
  inline `{% if %}` in the cards. Cosmetic only.
- The `title` attribute is the only place the taxonomy descriptions appear on filters (the
  contract allows it); it is poorly exposed on touch devices. A visible help line would
  be a small follow-up if wanted.
- `.gitignore` was not changed: the build writes only to `--output`, so nothing needed
  ignoring.
- The environment header of my session named `.worktrees/gallery-detect` as primary
  directory; I worked only in `.worktrees/gallery-site` as the task says.

### CLARIFICATION NEEDED

None.
