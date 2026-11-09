---
id: TASK-007
title: Redesign the site after the mockups
status: open             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/gallery/main.md
subrepo: gallery
issue:
assignee: coder
model:
created: 2026-11-09
updated: 2026-11-09
---

# TASK-007: Redesign the site after the mockups

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

Rebuild the home page and the template page so they match the mockups of
`subrepos/gallery/design/`, with the star button, the full file tree on every card and the
copy button, and remove giscus, the `main.md` preview and the "Detected information" block.

## Context

Read, in the specs repo:

- `subrepos/gallery/main.md`: Goal (principles), Scope, the Decisions of 2026-11-09,
  § Design tokens.
- `subrepos/gallery/design/README.md`, then open `design/index.html` and
  `design/template.html` in a text editor: their CSS and markup are the starting point of
  `site/assets/site.css` and of the Jinja templates. The logo is `design/logo.svg`.
- `architecture.md`: ADR-002 (everything from a template is escaped), ADR-006 (star link),
  ADR-007 (no third-party origin, CSP).

The build (`src/gallery/build.py`) already merges each entry with `detected.json` and
`votes.json`; adapt it, do not rewrite it. `votes.json` already carries `discussion_url`
per id; build ignores it today. The tree comes from `gallery.tree.render_tree` (TASK-006,
merged before this task starts).

No browser here: the look is reviewed by the orchestrator against the mockups. Give the
reviewer the built site in the scratch dir (Result: its path).

## Input contract

```yaml
repo: /home/omni/specfirst/gallery
worktree: /home/omni/specfirst/.worktrees/gallery-site-redesign
branch: task/007-site-redesign
depends_on: [TASK-006]
inputs:
  - /home/omni/specfirst/specsfirst-specs/subrepos/gallery/design/index.html
  - /home/omni/specfirst/specsfirst-specs/subrepos/gallery/design/template.html
  - /home/omni/specfirst/specsfirst-specs/subrepos/gallery/design/logo.svg
  - src/gallery/build.py
  - src/gallery/tree.py
  - site/
  - site.yml
scope:
  write:
    - site/                    # base.html, index.html, template.html, assets/
    - site/assets/fonts/       # WOFF2 files and OFL.txt
    - src/gallery/build.py
    - src/gallery/votes.py     # only the rename of the site.yml key (see contract)
    - site.yml
    - tests/test_build.py
    - tests/test_votes.py      # only the rename of the site.yml key
    - tests/helpers.py
    - pyproject.toml           # remove markdown-it-py
    - uv.lock
  read_only:
    - src/gallery/tree.py
    - src/gallery/detect.py
    - src/gallery/registry.py
    - src/gallery/validate.py
    - taxonomy.yml
    - schema/
    - templates/
    - .github/
    - CONTRIBUTING.md
checks:
  - uv sync
  - uv run pytest
  - uv run ruff check .
  - uv run python -m gallery.validate
  - detect, votes (GITHUB_TOKEN="$(gh auth token)") and build run locally into the scratch
    dir, as in publish.yml; the built site stays there for review
  - grep of the built site: no `giscus`, no `googleapis`, no `gstatic`, no `style="`, no
    inline `<script>` without `src`
```

## Output contract

```yaml
artifacts:
  - site/base.html
  - site/index.html
  - site/template.html
  - site/assets/site.css
  - site/assets/site.js
  - site/assets/fonts/ (WOFF2 + OFL.txt)
  - src/gallery/build.py
  - site.yml
  - tests
contract:
  general:
    - Markup and CSS follow the mockups (layout, sizes, spacing, order of elements). Where
      they differ from this contract, this contract wins.
    - Every color is a token of main.md § Design tokens, light by default, dark under
      `@media (prefers-color-scheme: dark)`; `<meta name="color-scheme" content="light dark">`
      stays.
    - Fonts: Inter 400/500/600 and JetBrains Mono 400/500, Latin subset, WOFF2, served from
      `assets/fonts/` with `font-display: swap`, plus the SIL OFL 1.1 text as
      `assets/fonts/OFL.txt`. Download them once from an official source (the projects'
      GitHub releases, or the @fontsource packages on the npm registry) and say which in
      Result. Fallbacks: `system-ui, sans-serif` and `ui-monospace, monospace`.
    - CSP meta, exactly: `default-src 'none'; script-src 'self'; style-src 'self';
      font-src 'self'; img-src 'self'; base-uri 'none'; object-src 'none'; form-action 'none'`.
      Hence no `style` attribute anywhere: tag colors are classes `.c0`–`.c5`.
    - The logo is `design/logo.svg`'s path, inlined in the pages (`aria-hidden="true"`, color
      `var(--accent)` through `currentColor`), next to the text "SpecsFirst".
    - Every interactive element is a real `<a>` or `<button>` with a visible focus ring and a
      target of at least 40 × 40 px (filter buttons and copy button included, unlike the
      mockups). Layout correct from 320 px wide, no horizontal page scroll (only the tree
      and the command scroll inside their block).
    - Without JavaScript every page is complete: all cards visible, the count readable; the
      filter buttons and the copy button are rendered `hidden` and shown by the script.
    - Everything coming from an entry or from detect is escaped by Jinja autoescape; links
      to a template's forge only when the url is https (existing `_is_https`), with
      `rel="nofollow noopener noreferrer"`.
    - `site.yml`: the key `giscus` becomes `discussions` (same four fields); `tagline` is
      removed; `load_site_config` and `votes.py` read the new key. Build adds
      `submit_url = <repo_url>/issues/new?template=submit-template.yml`.
  view:
    - Tags shown = the entry's `kind` first, then its resolved tags (existing
      `resolved_tags`), without duplicates.
    - Tag color index = position of the name in `taxonomy.yml` (kinds in file order, then
      tags in file order, deprecated ones included), modulo 6, rendered as class `c<i>`.
    - License: the SPDX id; `None` with class `warn` when detect found no license file;
      `Unknown` when a license file exists but is not identified; `Unknown` (muted) when
      detection failed or did not run.
    - Updated: `<time datetime="YYYY-MM-DD">` showing e.g. `Nov 6, 2026` (English month
      abbreviation, no zero padding, independent of the locale); `Unknown` (muted) when
      absent.
    - Source: link to the entry's url, text = the url without `https://` and without a
      trailing `/`, followed by ` ↗` (the arrow `aria-hidden="true"`).
    - Stars: `votes` of votes.json (0 when absent). Star link `href` = `discussion_url`
      when present and starting with `https://github.com/`, else
      `<repo_url>/discussions`. `target="_blank" rel="noopener"`, `title="Star on GitHub:
      opens the discussion, react with 👍"`, `aria-label="Star <name> on GitHub, N star(s)"`.
      Class `on` (filled star, star colors) when N > 0, outline otherwise.
    - Tree: `render_tree(detected.tree)` text in `<pre>` (`overflow-x: auto`,
      `tabindex="0"`), monochrome, then the summary line; when `tree_truncated`, add
      "Only the first 500 entries are listed." under it. When detection failed or did not
      run, the tree block holds one muted line instead: "File tree unavailable: <reason>."
      (reasons from the existing `ERROR_REASONS`, or "not analysed yet").
    - Order on the home page: stars descending, then last update descending (unknown
      last), then name, then id (`vote_order` updated).
    - Removed: the preview and its Markdown renderer (and the `markdown-it-py`
      dependency), tools, language, commit, the notice paragraph, the kind/tag fieldsets,
      the sort select, the footer, the giscus script and section.
  home_page:
    - Header: logo + "SpecsFirst" (link to the home) on the left; on the right a "GitHub"
      link (repo_url) and a "Submit a template" button-styled link (submit_url).
    - Intro, max 640 px: `<h1>Spec templates for agent-driven projects.</h1>` and the
      paragraph of the brief: "A specs repo holds what a project is about (goals,
      conventions, tasks), written for the humans and AI agents who build it. Compare the
      structures below, star the ones you like, and start yours with a single command."
    - Filter bar: `role="group" aria-label="Filter by tag"`, a button "All" then one button
      per kind/tag used by at least one card, in taxonomy order (dot + name),
      `aria-pressed`. On the right, `<p id="count" aria-live="polite">`: "N templates,
      sorted by stars" ("1 template, …"), correct without JS.
    - Grid `repeat(auto-fill, minmax(min(100%, 440px), 1fr))`. Each card is an `<article>`
      with, in order: `<h2>` title linking to `templates/<id>/`, the one-line description,
      the star link (top right, star + count); the `<dl>` License / Updated / Source / Tags;
      the tree block.
    - No card: "No template has been published yet."
  template_page:
    - Nav: logo + "SpecsFirst" on the left, "← All templates" on the right, both to the home.
    - One card, max 760 px: `<h1>` name + description, star link "☆ Star | N" as in the
      mockup on the right; the same `<dl>`; the tree; the command block.
    - Command block only when detect found a `copier.yml` (existing `copier_command`, with
      `my-project` instead of `<name>` everywhere): dark block, `<span aria-hidden>$ </span>`
      prompt not copied, the command in `<code>`, and a 40 × 40 transparent square
      `<button type="button" aria-label="Copy command">` with the two-sheets icon, which
      shows a ✓ for 1.5 s after a successful copy (aria-label "Copied" meanwhile).
  javascript:
    - `site.js`, plain ES5-style like today, only two features: the tag filter (single
      selection, shows/hides cards by their `data-tags` JSON, updates `aria-pressed` and
      the count text) and the copy button (only when `navigator.clipboard` exists, else it
      stays hidden). No URL state, no sorting, no storage. Page data read as attributes,
      written with `textContent` only.
  tests:
    - Update test_build.py to the new markup and drop the obsolete tests (preview, giscus,
      tools, sort and kind controls). Keep or adapt the security tests (escaping of hostile
      names, descriptions, tree names, urls; https-only links).
    - New tests: tag order, used-tags-only filter, color class from taxonomy position; star
      on/off, href fallback and aria-label plural; license None/Unknown; date format;
      source text; order stars → updated → name; tree block from a detect list and the
      unavailable message; command shown only with copier and using my-project; CSP meta
      exact; no `style=` attribute and no third-party URL in the output; filter and copy
      button rendered `hidden`; fonts and OFL.txt copied to the output.
    - A test computing WCAG contrast of the token pairs of main.md § Design tokens (text,
      muted, accent, warn on bg and card; term text and prompt on term-bg; accent-text on
      accent), parsed from site.css, ≥ 4.5 in both themes.
returns: the branch with its commit(s), the path of the built site in the scratch dir, and
  the filled Result section
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
