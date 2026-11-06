---
id: TASK-002
title: Detect template information safely
status: done             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/gallery/main.md
subrepo: gallery
issue:
assignee: coder
model: anthropic/claude-sonnet-5-5
created: 2026-11-06
updated: 2026-11-06
---

# TASK-002: Detect template information safely

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

Add `python -m gallery.detect`, which fetches every template of the registry and writes
`detected.json`, treating every template as hostile.

## Context

Read `subrepos/gallery/main.md`, and in `architecture.md` ADR-002 (the security rules of
this task) and § Data / interfaces (the exact format of `detected.json`).

TASK-001 is still being corrected on its own branch. It touches only `registry.py`,
`validate.py` and their tests. **Do not modify them.** Use `load_registry` and
`entry.fields` as they are; read `tests/helpers.py` for how to build a registry in a test.

## Input contract

```yaml
repo: /home/omni/specfirst/gallery
worktree: /home/omni/specfirst/.worktrees/gallery-detect
branch: task/002-detect
depends_on: [TASK-001]
inputs:
  - src/gallery/registry.py
  - tests/helpers.py
scope:
  write:
    - src/gallery/detect.py
    - tests/test_detect.py
  read_only:
    - src/gallery/registry.py
    - src/gallery/validate.py
    - tests/helpers.py
    - tests/test_registry.py
    - tests/test_validate.py
    - pyproject.toml
    - uv.lock
    - taxonomy.yml
    - schema/
    - templates/
checks:
  - uv sync
  - uv run pytest
  - uv run ruff check .
  - uv run python -m gallery.detect --output <scratch dir>/detected.json
```

The standard library and git are enough: no new dependency. If you think one is needed,
stop and say so.

## Output contract

```yaml
artifacts:
  - src/gallery/detect.py
  - tests/test_detect.py
contract:
  - `python -m gallery.detect [--root DIR] --output FILE [--jobs N]` writes the JSON of
    architecture.md, with the keys sorted and ids in order, and exits 0 even if every
    template failed. It exits non-zero only when the registry cannot be loaded or the
    output cannot be written. One template's failure never affects another.
  - Fetching: `git init --bare` in a fresh temporary directory, then `git fetch --depth 1
    --no-tags --filter=blob:limit=1m origin <ref or HEAD>`, read from FETCH_HEAD. For an
    entry with `path`, drop `--depth 1` and use `--filter=blob:none`, so that last_update is
    that of the last commit touching the path.
  - Never run a command that writes a working tree: only object reads such as `git
    ls-tree`, `git cat-file`, `git log`. No checkout, no submodule, no LFS. Lazy fetching of
    missing blobs is disabled; a missing blob counts as absent.
  - The environment of every git process is an allowlist (PATH, LANG, a temporary HOME),
    so no token of the runner reaches git or a template. It sets GIT_TERMINAL_PROMPT=0,
    GIT_CONFIG_NOSYSTEM=1, GIT_CONFIG_GLOBAL=/dev/null, and no credential helper, hooks or
    fsmonitor apply.
  - Only the https protocol is allowed, including on redirects (`protocol.allow=never`,
    `protocol.https.allow=always`). Tests inject the extra allowed protocol as a parameter
    of an internal function; the command line offers no way to widen it.
  - Limits, as named constants: 120 s per git command, 200 MB on disk per template (watched
    during the fetch; the process is killed beyond it), 500 tree entries, 64 KB of preview.
  - The registry is not trusted either: an entry whose url is not https, whose path is
    absolute or contains `..`, or whose ref starts with `-` fails with `invalid_entry`, and
    nothing is fetched.
  - `error` is one of a closed list (`timeout`, `too_large`, `unreachable`, `ref_not_found`,
    `path_not_found`, `invalid_entry`). git's stderr is never published: it can contain
    credentials or remote text. It may go to the program's own log only after removing
    CR, LF and any leading `::`, because it runs in GitHub Actions.
  - All detection is relative to `path` when set. Files are looked up in the tree of that
    commit, with their git mode.
  - license: the file LICENSE, LICENCE or COPYING, with or without an extension, in `path`,
    then at the repo root. An SPDX id comes from distinctive phrases of the text, for at
    least MIT, Apache-2.0, BSD-2-Clause, BSD-3-Clause, GPL-2.0, GPL-3.0, LGPL-3.0, AGPL-3.0,
    MPL-2.0, ISC, Unlicense, CC0-1.0, CC-BY-4.0, CC-BY-SA-4.0. A file not identified gives
    "unknown", no file gives null. GPL, LGPL and AGPL must not be confused with one another.
  - copier: true when `copier.yml` or `copier.yaml` exists as a regular file.
  - tools: the sorted subset found, from one table in the module: CLAUDE.md, AGENTS.md,
    GEMINI.md, .opencode/, .claude/, .cursor/, .cursorrules, .windsurfrules,
    .aider.conf.yml, .github/copilot-instructions.md.
  - last_update: committer date as an ISO 8601 UTC string ending in Z. commit: full sha.
  - tree: sorted paths relative to `path`, directories ending in `/`, at most 500 entries,
    with tree_truncated true beyond that. Symlinks and submodules are listed by name,
    never followed.
  - preview: the content of `main.md` only if it is a regular file (mode 100644 or 100755),
    never through a symlink; otherwise null. Cut at 64 KB on a character boundary; invalid
    UTF-8 is replaced, never fatal.
  - Tests run offline, with real git repos built in tmp_path. They cover: a normal repo;
    `main.md` as a symlink to a file outside the repo (preview null, target absent from the
    output); `ref` as a tag and as a commit; `path` with last_update of that path; a
    nonexistent ref, path and repo; invalid entries; each license, including GPL vs AGPL;
    truncated tree; non-UTF-8 and oversized preview; a timeout and a disk overflow (limits
    made small through parameters); one failing template next to a working one; the
    git environment holding no runner token (for example GITHUB_TOKEN set in the test);
    and no working tree created.
  - On the real registry (network), the check gives status ok for both entries, with
    CC0-1.0 and copier true for research-thesis, and a preview for both. Put the summary of
    this run in Result.
returns: the branch with its commit(s), and the filled Result section
```

## Attempts

> Filled in by the orchestrator, one line per (re)launch.

- 2026-11-06 #1: done. Review reproduced two defects, fixed by the orchestrator in a separate
  commit, with tests failing before and passing after: `https://[bad` raised out of
  `detect_entry` and stopped the whole run; LGPL-2.1 gave `unknown`, contrary to the Result.
  Review also ran `path` on real GitHub and GitLab repos (blobs fetched by id work on both),
  and 15 real license texts from the GitHub API.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

Done. Branch `task/002-detect`, one commit `be2a816`
("feat: add detect module fetching templates into bare repos without checkout").
Checks: `uv sync` ok, `uv run pytest` 113 passed (79 new in `tests/test_detect.py`,
about 9 s, offline), `uv run ruff check .` clean, and
`uv run python -m gallery.detect --output <scratch>/detected.json` exits 0.

Real registry run (network, 1.5 s): both entries `status: ok`.
- research-thesis: license `CC0-1.0`, copier `true`, tools `[]`, commit
  `a953a18c...`, last_update `2026-11-06T14:21:49Z`, 20 tree entries, preview 1215 chars.
- dev-studio-specs-template: license `null` (no license file in that repo), copier `false`,
  tools `[]`, commit `c58cc2d6...`, last_update `2026-11-03T16:06:00Z`, 14 tree entries,
  preview 1378 chars.

Design points beyond the letter of the contract:
- Lazy fetching is disabled with the env var `GIT_NO_LAZY_FETCH=1` (git 2.43 here has no
  `--no-lazy-fetch` option). The test fails if that variable is set to 0.
- Path entries: the history fetch is blobless as specified, so LICENSE and `main.md` blobs
  are absent. After resolving the tree, the module explicitly fetches only the 1 to 3 blobs
  it needs (`fetch --filter=blob:none origin <blob sha>...`, same timeout and disk watch).
  If the server refuses, those blobs stay absent (license/preview null or unknown).
- The git environment is PATH, LANG, a temporary HOME, plus the fixed GIT_* settings of the
  contract and `LC_ALL=C` (stable messages, to classify ref_not_found vs unreachable from
  stderr, which is never published) and `GIT_LITERAL_PATHSPECS=1`.
- Extra hardening of `invalid_entry`: a `ref` containing characters git forbids in ref names
  (`:` would make it a refspec, `*` a pattern, plus whitespace/control chars), a path with a
  Windows drive or backslash, a non-string field.
- `commit` is the fetched commit (FETCH_HEAD); with `path`, `last_update` is the date of the
  last commit touching it, so the two can differ.
- License file names are matched case-insensitively (`license.md` works) and only a regular
  file is read; a symlinked or unreadable license gives `"unknown"`. I also recognise
  LGPL-2.1 (not required) because the GNU family is identified from its earliest title.
- An unexpected internal exception in one template is contained and reported as
  `unreachable` (the error list is closed), with the exception type logged.

### Artifacts produced

- `src/gallery/detect.py` (branch `task/002-detect`, worktree
  `/home/omni/specfirst/.worktrees/gallery-detect`)
- `tests/test_detect.py`
- scratch: `tasks/.exchange/tmp/task-002/detected.json` (output of the real run)

### Opinion / concerns

- The contract's "path entries: blob:none, lazy fetch disabled" is self-contradictory for
  license and preview; the explicit blob fetch above is my resolution. It relies on the
  forge allowing a fetch by object id (GitHub and GitLab do); I could only exercise it on
  local repos, not on a real remote with `path`, since neither registry entry has one.
- Exit 0 when every template fails also hides a missing `git` binary or a broken runner
  (all templates `unreachable`). The log says so, but publish.yml may want to warn when
  zero templates are ok.
- The closed error list has no "internal" value; mapping bugs to `unreachable` is slightly
  misleading. Consider adding one in a later task if build wants to tell them apart.
- Disk and timeout limits are per template and per git command; the process group is
  killed. A repo with a 200 MB history still costs up to 120 s per command, so
  `--jobs` times that bounds the workflow; worth a job-level timeout in publish.yml.
- Slip, no impact: my first scratch experiment ran in the specs repo root by mistake
  (a failed `cd` was followed by commands) and created `src/`, `b*/` there. I deleted
  exactly those; `git status` of the specs repo shows only this task file modified.

### CLARIFICATION NEEDED

None.
