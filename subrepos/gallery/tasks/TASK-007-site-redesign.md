---
id: TASK-007
title: Redesign the site after the mockups
status: done             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/gallery/main.md
subrepo: gallery
issue:
assignee: coder
model: anthropic/claude-sonnet-5-5
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
merged). TASK-008 (merged) added `src/gallery/discussions.py`, which reads `site.giscus` like
votes: the key rename covers both. TASK-009 (merged) added the issue form
`.github/ISSUE_TEMPLATE/submit-template.yml`, the target of the Submit button, and its
`submit.py`, which this task does not touch.

No browser here: the look is reviewed by the orchestrator against the mockups. Give the
reviewer the built site in the scratch dir (Result: its path).

## Input contract

```yaml
repo: /home/omni/specfirst/gallery
worktree: /home/omni/specfirst/.worktrees/gallery-site-redesign
branch: task/007-site-redesign
depends_on: [TASK-006, TASK-008]
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
    - src/gallery/discussions.py   # idem
    - tests/test_discussions.py    # idem
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
      removed; `load_site_config`, `votes.py` and `discussions.py` read the new key. Build adds
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

- 2026-11-09 #1 (Claude Sonnet subagent): done. Review re-ran pytest (507), ruff, validate, a
  build from the coder's data, and the scan of the output (no third-party URL, no inline
  script or style). One orchestrator fix in a separate commit: the coder had flagged that the
  Latin subset of JetBrains Mono lacks the box-drawing glyphs; fontTools confirmed it, and the
  full fonts of release v2.304 have them at the same 600 advance, so they replace it. Visual
  check against the mockups left to the operator (no browser here).

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

Done, branch `task/007-site-redesign` (4 commits on top of `origin/main`, nothing pushed):

- `6eba52c` feat(site): serve Inter and JetBrains Mono from the site itself
- `061a50f` refactor(site): rename the giscus key to discussions and drop the tagline
  (site.yml, build, votes, discussions and their tests; the old templates were adjusted
  minimally so this commit builds)
- `b6be980` feat(site): redesign the home and template pages after the mockups
- `0afd34b` chore: move markdown-it-py to the dev dependencies

Checks (all run in the worktree): `uv sync` ok; `uv run pytest` 507 passed; `uv run ruff
check .` clean; `uv run python -m gallery.validate` exit 0; detect (2/2 ok), votes (read-only,
`GITHUB_TOKEN="$(gh auth token)"`, 2 templates with a Discussion, 1 star each) and build run
into the scratch dir as in publish.yml. Grep of the built site: no `giscus`, `googleapis`,
`gstatic`, no `style="`, no inline `<script>` (also covered by tests). `discussions.py` was
never run (not even `--dry-run`: not needed).

**Built site for review:** `/home/omni/specfirst/specsfirst-specs/subrepos/gallery/tasks/.exchange/tmp/task-007/site/`
(`index.html`, `templates/<id>/index.html`, `assets/`), built from `data/detected.json` and
`data/votes.json` next to it. No browser here: the look is NOT visually verified.

**Fonts** (untrusted data, downloaded once with curl into a fresh dir, never executed): the
npm tarballs of `@fontsource/inter` 5.3.0 and `@fontsource/jetbrains-mono` 5.3.0 from
`registry.npmjs.org` (sha256 `02034af8...8fd6` and `1bbea47d...0bba`), files
`files/inter-latin-{400,500,600}-normal.woff2` and
`files/jetbrains-mono-latin-{400,500}-normal.woff2`, copied unmodified to
`site/assets/fonts/`. `OFL.txt` = both copyright lines (Inter Project Authors, JetBrains
Mono Project Authors) + the OFL 1.1 text from the packages' LICENSE files (identical bodies).

### Artifacts produced

- `site/base.html`, `site/index.html`, `site/template.html`, new `site/_parts.html` (macros:
  logo, star, meta `<dl>`, tree block), `site/assets/site.css`, `site/assets/site.js`
- `site/assets/fonts/` (5 WOFF2 + `OFL.txt`)
- `src/gallery/build.py` (preview renderer and markdown code removed; new view model, tag
  colors, filters, star link, date/license/source/tree/command; fonts copied to the output)
- `site.yml` (`discussions:` key, no `tagline`), `src/gallery/votes.py`,
  `src/gallery/discussions.py`, `tests/test_votes.py`, `tests/test_discussions.py` (key rename)
- `tests/test_build.py` rewritten (62 test functions, 507 tests in the suite)
- `pyproject.toml`, `uv.lock`

### Opinion / concerns

1. **markdown-it-py is not removed, it moves to the dev group.** `tests/test_submit.py`
   (TASK-009, outside this task's scope) imports `markdown_it` to check that the submission
   comment is inert Markdown. Removing the dependency entirely would break the suite; the
   runtime dependency is gone (the site no longer uses it). If you want it fully removed,
   `test_submit.py` needs a different check (a task of its own).
2. **Box-drawing characters are not in the Latin subset** of JetBrains Mono (nor are `↗`,
   `←`, `✓`, which Inter's Latin subset lacks too). The tree's `├── │ └──` therefore render in
   the `ui-monospace` fallback, exactly as on the mockups served by Google Fonts (same
   subsets). Alignment depends on the visitor's fallback font; worth a look in the review.
   Including U+2500-257F would mean a custom subset (not an official file).
3. **Colors that are not tokens in the mockups** (`--term-border`, `--term-icon`, hover
   `#2F41C0`, `rgba(255,255,255,.06)`): replaced with `color-mix()` of tokens (accent/text
   and term-text over transparent). A test asserts every hex in `site.css` is a token or one
   of the six palette colors and that the tokens equal main.md's table.
4. **Gaps the contract leaves open, my choices:** (a) a tag not in `taxonomy.yml` (shown as
   is, e.g. a typo) gets color `c0` and is last in the filter; (b) a detect `failed` whose
   `error` is not in `ERROR_REASONS` shows "File tree unavailable: the analysis failed."
   (the contract only lists the known reasons and "not analysed yet"); (c) dates use the
   UTC day of `last_update`; (d) the focusable `<pre>` and the command line carry
   `role="region"` + `aria-label` so the focus ring lands on a named element; (e)
   `SiteConfig.contributing_url` was removed (no footer any more), `submit_url` is a property.
5. **40 px targets:** nav links, brand, filters, star and copy are real 40 px boxes; the card
   title and the Source link, being inline text, get an invisible `::after` hit area of 40 px
   height (overlapping only non-interactive neighbours). Verify visually.
6. `CONTRIBUTING.md` (read-only here) is consistent with the new site (it already says there
   is no preview); `architecture.md` ADR-001 still says "filters and sorting are plain
   JavaScript": sorting is now build-time only, worth a one-word edit by the orchestrator.

### CLARIFICATION NEEDED

None blocking. Only item 1 (markdown-it-py kept as a dev dependency) deviates from the
contract's "remove markdown-it-py".
