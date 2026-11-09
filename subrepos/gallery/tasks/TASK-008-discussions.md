---
id: TASK-008
title: Create each template's Discussion in the publish workflow
status: done             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/gallery/main.md
subrepo: gallery
issue:
assignee: coder
model: anthropic/claude-sonnet-5-5
created: 2026-11-09
updated: 2026-11-09
---

# TASK-008: Create each template's Discussion in the publish workflow

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

Add `python -m gallery.discussions`, which creates the missing Discussion of each registry id,
and run it in a new job of `publish.yml` before the build, so every star button opens a
Discussion.

## Context

Read `architecture.md` ADR-004 (title rule, oldest counts) and ADR-006 (this job), and
`subrepos/gallery/main.md` § Discussions configuration and § Operator setup (item 8).

`src/gallery/votes.py` already lists every Discussion of the category with GraphQL, with a
strict, all-or-nothing reading, an injected transport, no redirect followed and the token
never logged. Reuse it (extract what you need into shared functions of `votes.py`); do not
duplicate the HTTP or the page-checking code. Read the configuration exactly as votes does
(`load_site_config`, today under the `site.yml` key `giscus`): TASK-007, which runs after this
task, renames that key for votes and discussions together. Do not touch `site.yml` or
`build.py`.

TASK-009 (merged) added `submit.yml` and `src/gallery/submit.py`; they are unrelated.

You must not create real Discussions: the operator's first publish does. Against the real API,
run only `--dry-run`.

## Input contract

```yaml
repo: /home/omni/specfirst/gallery
worktree: /home/omni/specfirst/.worktrees/gallery-discussions
branch: task/008-discussions
depends_on: []
inputs:
  - src/gallery/votes.py
  - tests/test_votes.py
  - .github/workflows/publish.yml
  - tests/test_publish_workflow.py
  - site.yml
scope:
  write:
    - src/gallery/discussions.py
    - src/gallery/votes.py          # extract shared helpers only; behaviour unchanged
    - tests/test_discussions.py
    - tests/test_votes.py           # only if the extraction requires it
    - .github/workflows/publish.yml
    - tests/test_publish_workflow.py
  read_only:
    - site.yml
    - src/gallery/build.py
    - src/gallery/registry.py
    - src/gallery/detect.py
    - taxonomy.yml
    - templates/
    - pyproject.toml
    - uv.lock
checks:
  - uv sync
  - uv run pytest
  - uv run ruff check .
  - uvx --from actionlint-py actionlint .github/workflows/publish.yml
  - GITHUB_TOKEN="$(gh auth token)" uv run python -m gallery.discussions --dry-run
```

The standard library is enough: no new dependency.

## Output contract

```yaml
artifacts:
  - src/gallery/discussions.py
  - tests/test_discussions.py
  - .github/workflows/publish.yml
  - tests/test_publish_workflow.py
contract:
  command:
    - `python -m gallery.discussions [--root DIR] [--dry-run]`. Token from GITHUB_TOKEN only,
      as votes. Ids, names, descriptions and urls from `load_registry`; repo, repo_id and
      category_id from site.yml, read as votes reads them; the gallery's base url derived
      from the repo: `https://<owner lowercased>.github.io/<name>/` (so
      `https://specsfirst.github.io/gallery/`).
    - Lists the Discussions of the category with votes' query and checks; an id is missing
      when no Discussion has exactly that title.
    - For each missing id, in id order, runs exactly:
      mutation($repo:ID!,$cat:ID!,$title:String!,$body:String!){createDiscussion(input:{repositoryId:$repo,categoryId:$cat,title:$title,body:$body}){discussion{url}}}
      with title = the id and body:
        Star **<name>** by reacting with 👍 to this post. One GitHub account, one star; the
        gallery counts them every few hours.

        > <description, with every `@` written `@&#8203;` so it mentions no one>

        - Gallery page: <url>templates/<id>/
        - Source: <entry url>
    - At most `MAX_CREATE = 10` creations per run (named constant): the rest wait for the
      next run, and the output says how many are left.
    - `--dry-run` sends no mutation and prints the ids it would create.
    - Prints one line per created id with its Discussion url, and nothing from other
      Discussions (titles of strangers are never logged).
    - Failures: listing fails → exit 1, nothing created. A creation fails → stop at once,
      exit 1, the ones already created stay (the next run skips them). Same error rules as
      votes: fixed text plus at most the HTTP status, a non-empty `errors` or a
      `discussion.url` not starting with `https://github.com/` is a failure.
  workflow:
    - New job `discussions`, first in publish.yml: permissions exactly `contents: read` and
      `discussions: write`, `timeout-minutes: 10`; checkout with persist-credentials false,
      setup-uv, setup-python and `uv sync --frozen` as the build job; one step
      `uv run python -m gallery.discussions` with `GITHUB_TOKEN: ${{ github.token }}` in its
      env.
    - Job `build`: `needs: discussions` and `if: ${{ !cancelled() }}`, so a failed creation
      leaves a red job but still publishes. Its permissions and steps are unchanged.
    - Triggers, top-level permissions, concurrency and the deploy job are unchanged.
  tests:
    - Offline, with an injected transport: nothing missing → no mutation; two missing →
      two mutations in id order with the exact variables and body; a Discussion with the
      title in another case does not count as present; MAX_CREATE respected; dry run sends
      no mutation; each failure above with the exit code; token and remote text never in
      stdout/stderr.
    - test_publish_workflow.py: the new job (permissions, token only on its run step, no
      `${{` inside `run:`), build's `needs` and `if`, and the token now on exactly two steps
      (votes and discussions).
returns: the branch with its commit(s), and the filled Result section
```

## Attempts

> Filled in by the orchestrator, one line per (re)launch.

- 2026-11-09 #1 (Claude Sonnet subagent, parallel to TASK-006): done. Review re-ran pytest,
  ruff, actionlint and the real `--dry-run` (2 Discussions already present, created by giscus).
  Two orchestrator fixes in separate commits: the body sentence was split by a newline the
  spec's wrapping had put in it (GitHub renders it as a line break), and the job now runs on
  `refs/heads/main` only, since `workflow_dispatch` can target any branch.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

Done on branch `task/008-discussions`, two commits:

- `refactor(votes): expose the Discussion listing and the GraphQL post` - extracted from
  `votes.py`: `post_checked` (status/size/JSON checks), `list_discussions` (all pages,
  checked nodes), and made public `split_repo`, `site_config_path`, `malformed`.
  `fetch_votes` is now built on `list_discussions`; behaviour and `test_votes.py` unchanged
  (59 tests pass untouched).
- `feat(discussions): create each template's Discussion before the build` -
  `src/gallery/discussions.py`, `tests/test_discussions.py` (41 tests), new `discussions`
  job in `publish.yml` (first; `build` has `needs: discussions` and
  `if: ${{ !cancelled() }}`), `tests/test_publish_workflow.py` updated.

Checks: `uv sync` ok; `uv run pytest` 457 passed; `uv run ruff check .` clean;
`actionlint publish.yml` clean; `GITHUB_TOKEN="$(gh auth token)" uv run python -m
gallery.discussions --dry-run` -> exit 0, "would create 0, 2 already present, 0 left for
the next run" (the two templates already have a Discussion). No mutation was ever sent to
the real API.

Output: `created <id> <url>` / `would create <id>` on stdout; a summary line on stderr.
Dry run also respects `MAX_CREATE`. Exit codes: 2 for token/config, 1 for API failures.

### Artifacts produced

- `src/gallery/discussions.py`, `tests/test_discussions.py`
- `src/gallery/votes.py` (shared helpers extracted), `.github/workflows/publish.yml`,
  `tests/test_publish_workflow.py`

### Opinion / concerns

- The body quotes the description line by line (`> ` on each line) so a multi-line
  description cannot leave the quote; identical to the spec for one line. I also apply the
  `@` -> `@&#8203;` escape to the template name (spec says description only): a name may
  contain `@user` too. Urls are left alone, since escaping would break the link.
- `workflow_dispatch` can be started on any branch by someone with write access, and the
  `discussions` job then reads that branch's registry. Not in scope; a job-level
  `if: github.ref == 'refs/heads/main'` would close it (the build job has the same
  exposure today).
- Item 8 of Operator setup (can `github-actions[bot]` create a Discussion in the
  Announcement-type category?) is still unverified: only the first real publish shows it.
- The code reads `site.giscus[...]` in one place (`discussions.main`); TASK-007's key
  rename touches that line and `votes.main`.

### CLARIFICATION NEEDED

None.
