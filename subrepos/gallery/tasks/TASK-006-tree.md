---
id: TASK-006
title: Render the file tree like `tree`
status: done             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/gallery/main.md
subrepo: gallery
issue:
assignee: coder
model: albert/deepseek-v4-flash-0731
created: 2026-11-09
updated: 2026-11-09
---

# TASK-006: Render the file tree like `tree`

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

Add `src/gallery/tree.py`, a pure function that turns detect's `tree` list into the exact text
of `LC_ALL=C tree` (without `-a`), with its summary line.

## Context

Read `subrepos/gallery/main.md` (§ Decisions of 2026-11-09 on the tree) and
`architecture.md` § Data / interfaces (`detected.json`, field `tree`).

Detect's `tree` is a sorted list of paths relative to the template root, with `/` after the
name of each directory (`git ls-tree -r -t`): every directory appears on its own, before its
content. It may be cut at 500 entries (`tree_truncated: true`), in which case a directory may
appear without all its content, or a file may appear without its parent directory entry.
Symlinks and submodules are plain names (never followed).

The redesign (TASK-007) shows this text on every card and template page. This task only adds
the function and its tests; it does not touch the site.

Reference: `SpecsFirst/template-research-thesis` (sibling checkout at
`/home/omni/specfirst/template-research-thesis`) gives, with `LC_ALL=C tree`:

```
.
├── LICENSE
├── README.md
├── analysis-code
│   ├── main.md
│   └── tasks
│       ├── TASK-002-pipeline-example.md
│       ├── TASK-003-analysis-example.md
│       └── board.md
├── conventions.md
├── copier.yml
├── environment.md
├── hypotheses
│   └── H1-short-name.md
├── lab
│   ├── protocols
│   │   └── short-name.md
│   └── tasks
│       ├── TASK-001-bench-example.md
│       └── board.md
└── main.md

6 directories, 14 files
```

## Input contract

```yaml
repo: /home/omni/specfirst/gallery
worktree: /home/omni/specfirst/.worktrees/gallery-tree
branch: task/006-tree
depends_on: []
inputs:
  - src/gallery/detect.py      # how `tree` is produced (_inspect, parse_ls_tree)
  - tests/test_detect.py
scope:
  write:
    - src/gallery/tree.py
    - tests/test_tree.py
  read_only:
    - src/gallery/detect.py
    - src/gallery/build.py
    - tests/
    - pyproject.toml
    - uv.lock
checks:
  - uv sync
  - uv run pytest
  - uv run ruff check .
  - the reference above, produced from the `tree` list that `python -m gallery.detect` gives
    for research-thesis (run detect into the scratch dir), compared byte for byte
```

The standard library is enough: no new dependency.

## Output contract

```yaml
artifacts:
  - src/gallery/tree.py
  - tests/test_tree.py
contract:
  api:
    - `render_tree(paths: Sequence[str]) -> RenderedTree`, where `RenderedTree` is a frozen
      dataclass with `text: str` (the lines, joined with "\n", no trailing newline),
      `directories: int`, `files: int`, and a property `summary: str`.
    - `summary` is `"X directories, Y files"` with tree's singulars: "1 directory",
      "1 file", "0 directories", "0 files".
    - Pure: no I/O, no logging, deterministic.
  rendering:
    - First line `.`; then each entry as `prefix + connector + name`, where the connector is
      `├── ` or `└── ` (last child) and the prefix adds `│   ` or `    ` per ancestor level,
      exactly as `tree` does (box-drawing characters U+251C, U+2514, U+2502, U+2500 and
      spaces, as in the reference).
    - Children of each directory sorted in byte order of their UTF-8 name (`LC_ALL=C`),
      files and directories mixed (no dirs-first).
    - The trailing `/` of directory entries is not shown.
    - Hidden entries: any entry whose own name starts with `.` is left out, with its whole
      subtree. Nothing else is filtered.
    - A path whose parent directory entry is missing (truncated listing) still shows under
      its parents, which are created as directories.
    - Counts exclude the root and hidden entries: a directory is an entry that had a
      trailing `/` or has children; everything else is a file.
    - Empty input gives text `.` and summary "0 directories, 0 files".
    - Robust to hostile names: empty segments, `.` and `..` segments, and duplicate paths are
      ignored; names are returned as given (escaping is the caller's job: Jinja autoescape).
      Control characters in a name (U+0000–U+001F, U+007F) are replaced with `?`, like
      `tree` does on a terminal, so a name cannot break a line.
  tests:
    - The research-thesis reference above, from its real detect list, byte for byte.
    - Byte order: `README.md` before `analysis-code`, `Z` before `a`, `_x` after `Z`,
      non-ASCII after ASCII.
    - Nesting at three levels with `│   ` and `    ` prefixes, a last directory with content.
    - Hidden: `.github/`, `.github/workflows/x.yml`, `.gitignore`, `a/.hidden` left out and
      not counted; `a.b` kept.
    - Truncated listing with a missing parent; empty input; singular/plural summaries;
      names with a newline or a tab.
returns: the branch with its commit(s), and the filled Result section
```

## Attempts

> Filled in by the orchestrator, one line per (re)launch.

- 2026-11-09 #1 (opencode, deepseek-v4-flash): done, one commit, trailers ok. Review re-ran
  pytest and ruff, and rendered the trees of a real detect run on both templates: identical to
  the mockups. Recursion depth is bounded by detect's 500-entry cap.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

Implemented `src/gallery/tree.py` with `render_tree(paths: Sequence[str]) -> RenderedTree`
(frozen dataclass: `text`, `directories`, `files`, `summary` property), pure and
deterministic. Added `tests/test_tree.py` (12 tests). Verified byte-for-byte against the
research-thesis reference from the real `python -m gallery.detect` list. All checks pass:
`uv sync`, `uv run pytest` (287 passed), `uv run ruff check .` (clean).

The reference check: built a local git repo of `template-research-thesis` and ran detect
into the scratch dir; detect produced the 20-entry `tree` list which `render_tree` turns
into exactly the `LC_ALL=C tree` text in the task spec (`6 directories, 14 files`). That
exact detect list and text are hardcoded in the tests, and an integration test re-runs
detect→render on a git repo with the research-thesis structure and asserts equality.

### Artifacts produced

- `src/gallery/tree.py` (new)
- `tests/test_tree.py` (new)
- Commit `a90c627` on branch `task/006-tree` in worktree
  `/home/omni/specfirst/.worktrees/gallery-tree`
- Scratch: `python -m gallery.detect` list and rendering verified in
  `.exchange/tmp/opencode/`

### Opinion / concerns

- `tree` is not installed in this environment (no sudo), so the reference was taken from
  the task spec and matched byte-for-byte; the algorithm (prefix `│   `/`    `, connectors
  `├── `/`└── `, byte-order sort, singular/plural summary) follows `tree` exactly.
- Sorting uses `name.encode("utf-8")` byte comparison to match `LC_ALL=C`; Python `bytes`
  compare lexicographically by unsigned byte value, which equals C-locale byte order.
- Hostile inputs (empty/`.`/`..` segments, duplicates, control chars) are handled; a name
  that appears as both file and directory is not defined by the spec — detect never
  produces it, and it is resolved defensively (promoted to a directory).
- The worktree branch `task/006-tree` is behind `origin/main` (4 commits); the commit is
  on top of the branch point, ready for the orchestrator to integrate.

### CLARIFICATION NEEDED
