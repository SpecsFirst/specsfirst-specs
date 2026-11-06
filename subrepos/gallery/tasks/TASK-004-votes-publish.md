---
id: TASK-004
title: Votes and publication workflow
status: done             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/gallery/main.md
subrepo: gallery
issue:
assignee: coder
model: anthropic/claude-sonnet-5-5
created: 2026-11-06
updated: 2026-11-06
---

# TASK-004: Votes and publication workflow

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

Add `python -m gallery.votes`, which writes `votes.json`, and `.github/workflows/publish.yml`,
which runs detect, votes and build on a schedule and publishes the site to GitHub Pages.

## Context

Read `subrepos/gallery/main.md`, and in `architecture.md` ADR-002, ADR-003, ADR-004 and
§ Data / interfaces (the exact format of `votes.json`).

Detect and build are merged. Their command lines are in the docstrings of
`src/gallery/detect.py` and `src/gallery/build.py`. `build.py` reads `votes` per id and
already counts a missing id as 0 votes. Do not modify either file.

Runs on the first publish only can prove the workflow: you cannot run Actions here. So the
workflow is checked with actionlint, with tests on its structure, and by running each of its
commands locally.

## Input contract

```yaml
repo: /home/omni/specfirst/gallery
worktree: /home/omni/specfirst/.worktrees/gallery-publish
branch: task/004-publish
depends_on: [TASK-002, TASK-003]
inputs:
  - src/gallery/build.py
  - src/gallery/detect.py
  - site.yml
  - .github/workflows/validate.yml
scope:
  write:
    - src/gallery/votes.py
    - tests/test_votes.py
    - tests/test_publish_workflow.py
    - .github/workflows/publish.yml
    - .gitignore
  read_only:
    - src/gallery/build.py
    - src/gallery/detect.py
    - src/gallery/registry.py
    - src/gallery/validate.py
    - tests/helpers.py
    - pyproject.toml
    - uv.lock
    - site.yml
    - taxonomy.yml
    - schema/
    - templates/
    - .github/workflows/validate.yml
checks:
  - uv sync
  - uv run pytest
  - uv run ruff check .
  - uvx --from actionlint-py actionlint .github/workflows/publish.yml
  - GITHUB_TOKEN="$(gh auth token)" uv run python -m gallery.votes --output <scratch dir>/votes.json
  - the commands of publish.yml, in order, run locally into a scratch directory
```

The standard library is enough for votes (`urllib`, `json`): no new dependency. If you think
one is needed, stop and say so.

## Output contract

```yaml
artifacts:
  - src/gallery/votes.py
  - tests/test_votes.py
  - tests/test_publish_workflow.py
  - .github/workflows/publish.yml
contract:
  votes:
    - `python -m gallery.votes [--root DIR] --output FILE` writes the JSON of architecture.md,
      with keys sorted and `generated_at` in the same format as detect. The owner, repo name
      and category_id come from the `giscus` mapping of site.yml; ids come from `load_registry`.
    - The token comes from the environment variable GITHUB_TOKEN only, with no command-line
      option. It goes only in the Authorization header of POST https://api.github.com/graphql.
      It is never logged.
    - This exact query, already run against the real API, with every page read through
      `pageInfo` and a named constant capping the number of pages:
      query($owner:String!,$name:String!,$cat:ID!,$after:String){repository(owner:$owner,name:$name){discussions(first:100,after:$after,categoryId:$cat,orderBy:{field:CREATED_AT,direction:ASC}){pageInfo{hasNextPage endCursor} nodes{title url reactions(content:THUMBS_UP,first:1){totalCount}}}}}
    - A Discussion counts only if its title is exactly an id of the registry (case-sensitive).
      Other titles are ignored and never logged. With two Discussions of the same title, the
      oldest one counts. An id with no Discussion is absent from the output.
    - All or nothing. A missing token, a network error, a timeout (named constant, 30 s), a
      status other than 200, a redirect, invalid JSON, a non-empty `errors`, a null repository,
      a missing field, a count that is not a non-negative int (bool excluded), a url not
      starting with `https://github.com/`, or going over the page cap all exit non-zero and
      write no file.
    - Redirects are never followed, because urllib would resend the Authorization header.
    - Error messages are fixed text plus at most the HTTP status: no remote text, since they
      run in GitHub Actions logs.
    - The output is written only after every page succeeded.
    - Tests run offline through an injected transport, with no socket. They cover: exact title
      match; case difference; duplicates; unknown titles; several pages; each failure above
      with no file written; a redirect not followed, using a real local HTTP server if the
      transport cannot show it; and the token absent from stdout and stderr.
  workflow:
    - Triggers are exactly push on main, schedule `17 */6 * * *`, and workflow_dispatch. Never
      pull_request or pull_request_target.
    - Top-level `permissions: {}`. `concurrency` with group `pages` and cancel-in-progress false.
    - Job `build`, with permissions contents read and discussions read, `timeout-minutes: 30`,
      in this order:
        1. actions/checkout@v4 with persist-credentials false.
        2. astral-sh/setup-uv@v5, actions/setup-python@v5 and `uv sync --frozen`, as validate.yml.
        3. `uv run python -m gallery.detect --output <data dir>/detected.json`, with no token
           in its environment.
        4. A `jq -e` step that fails with `::error::` when the registry has at least one entry
           and none has status ok.
        5. `uv run python -m gallery.votes --output <data dir>/votes.json`, the only step with
           `GITHUB_TOKEN: ${{ github.token }}` in its env.
        6. `uv run python -m gallery.build --detected ... --votes ... --output _site`.
        7. actions/upload-pages-artifact@v5 with path `_site`.
      The data dir is outside `_site`, so the JSON files are not published.
    - Job `deploy`, with `needs: build`, permissions pages write and id-token write,
      `timeout-minutes: 10`, environment `github-pages` with its url from
      `steps.deployment.outputs.page_url`, and one step actions/deploy-pages@v5 with
      id `deployment`. It has no checkout and runs no repository code.
    - upload-pages-artifact@v5 leaves out dotfiles, so `.nojekyll` is not published. This is
      fine: a Pages artifact is not processed by Jekyll. Do not work around it.
    - tests/test_publish_workflow.py parses the YAML (PyYAML reads the key `on` as True) and
      asserts each point of this workflow section that a parse can show: triggers, permissions
      per job, token only on the votes step, step order, timeouts, persist-credentials, no
      checkout in deploy.
    - `_site/` is git-ignored.
returns: the branch with its commit(s), and the filled Result section
```

## Attempts

> Filled in by the orchestrator, one line per (re)launch.

- 2026-11-06 #1: done (the coder's commit 9853784 was stamped as b779163). Review reproduced one
  defect, fixed by the orchestrator in a separate commit, with tests failing before and passing
  after: a bad status line, a truncated chunk or very deep JSON left a traceback, and BadStatusLine
  quotes remote text, which must stay out of Actions logs. Review also compared `fetch_votes` with
  `reactionGroups` on 93 real public Discussions of giscus/giscus (no mismatch, two with a 👍),
  and seven mutations of publish.yml each failed its tests.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

Done, one commit `9853784` on `task/004-publish` (not pushed).

Checks, real results:
- `uv sync`: ok.
- `uv run pytest`: 272 passed (the repo's earlier tests plus the new ones).
- `uv run ruff check .`: all checks passed.
- `uvx --from actionlint-py actionlint .github/workflows/publish.yml`: exit 0, no finding.
- `GITHUB_TOKEN="$(gh auth token)" uv run python -m gallery.votes --output <scratch>/votes.json`: exit 0 against the real API. The output is `{"generated_at": ..., "templates": {}}`: no Discussion exists yet, so every id is absent (expected).
- The `run` commands of publish.yml, extracted from the YAML and run in order from the worktree into a scratch dir (`RUNNER_TEMP`=scratch, `--output _site` redirected to the scratch dir): install, detect (2/2 ok, real clones), jq check, votes (real API, token via `gh auth token`), build (2 pages). All exit 0. The jq step was also run on a detected.json with only failed entries (prints `::error::`, exit 1) and on an empty registry (exit 0).
- Not checkable here: the real Actions run (checkout, setup actions, upload/deploy of Pages, the `github.token` permissions `discussions: read`, `actions/*@v5` of upload-pages-artifact and deploy-pages, which actionlint accepts). The first publish proves those. With a real Discussion present, only the offline tests and a local server cover the counting; the real API run returned zero Discussions.

Design notes:
- `votes.py`: transport signature `(body: bytes, token: str) -> (status, bytes)`, injected into `main(argv, transport)` and `fetch_votes`. Default `post_graphql` uses urllib with an opener whose redirect handler returns None, so a 3xx surfaces as an HTTPError status and is treated as a failure. The response is read with an 8 MiB cap (my addition, same all-or-nothing rule). `MAX_PAGES = 20`, `REQUEST_TIMEOUT = 30`.
- Registry ids come from `load_registry`; site config from `gallery.build.load_site_config` (not modified), falling back to the packaged `site.yml` like build does. Output written with `detect.write_output` (atomic, sorted keys); `generated_at` has detect's format.
- Exit codes: 2 for local problems (missing token, unreadable registry or site.yml, cannot write), 1 for any API failure. No file is written on any failure.
- Every node of every page is checked (title, url prefix, count), not only the matching ones, so one malformed foreign Discussion fails the run.
- Workflow data dir is `$RUNNER_TEMP/gallery-data` (outside `_site`); the detect step does `mkdir -p` first, since detect does not create the directory. The `runner` context is not available in job-level `env`, hence `$RUNNER_TEMP` in the shell.
- Tests: offline FakeTransport plus real local `ThreadingHTTPServer` for: redirect not followed and token not resent to the target (server records no request), real timeout, round trip header. Token and remote text absent from stdout/stderr are checked on success and on each failure.

### Artifacts produced

- `src/gallery/votes.py`
- `tests/test_votes.py`
- `tests/test_publish_workflow.py`
- `.github/workflows/publish.yml`
- `.gitignore` (adds `_site/`)

### Opinion / concerns

- Strict validation of foreign Discussions means a single malformed node (in practice impossible from GitHub) blocks all publication until the next run; consistent with the contract's "a missing field" rule, but worth knowing.
- The cron `17 */6 * * *` plus `push` on `main` both deploy; with `cancel-in-progress: false` runs queue, which is fine.
- `.nojekyll` is written by build but dropped by upload-pages-artifact@v5, as the contract says; left alone.
- The jq check lets an empty registry pass (zero entries), as specified.
- The Result of the real-API run is only a zero-vote document; the counting against real Discussions is unproven until the operator enables Discussions and one exists.

### CLARIFICATION NEEDED

None.
