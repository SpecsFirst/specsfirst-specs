---
id: TASK-009
title: Submit a template through an issue form
status: open             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/gallery/main.md
subrepo: gallery
issue:
assignee: coder
model:
created: 2026-11-09
updated: 2026-11-09
---

# TASK-009: Submit a template through an issue form

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

Let anyone submit a template by filling a GitHub issue form: an Action checks the repository,
writes `templates/<id>.yml`, opens the pull request "Closes #N", and comments on the issue
with the link or with clear errors. Update `CONTRIBUTING.md` to lead with the form.

## Context

Read `architecture.md` ADR-002 (templates are untrusted) and ADR-008 (this design, two jobs),
§ Data / interfaces (`submission.json`), and `subrepos/gallery/main.md` (Scope, Decisions,
§ Operator setup items 6 and 7).

Reuse, do not duplicate: `registry.load_entry` / `load_taxonomy`, the checks of
`validate.py` (you may expose a function returning the list of issues for one entry, with the
CLI's behaviour unchanged), and `detect.detect_entry` for reachability and the detected facts.

Everything in the issue is hostile: the body, the title, the URL. It must never reach a
shell through `${{ }}`, never be logged raw in Actions (`::` workflow commands), and never be
echoed into the comment or the pull request unescaped (mentions, images, HTML).

This task runs in parallel with TASK-006/007/008: do not touch `site/`, `build.py`,
`votes.py`, `publish.yml` or `site.yml`.

## Input contract

```yaml
repo: /home/omni/specfirst/gallery
worktree: /home/omni/specfirst/.worktrees/gallery-submit-form
branch: task/009-submit-form
depends_on: []
inputs:
  - src/gallery/registry.py
  - src/gallery/validate.py
  - src/gallery/detect.py
  - schema/template.schema.json
  - taxonomy.yml
  - CONTRIBUTING.md
  - .github/workflows/validate.yml
scope:
  write:
    - .github/ISSUE_TEMPLATE/submit-template.yml
    - .github/workflows/submit.yml
    - src/gallery/submit.py
    - src/gallery/validate.py       # expose per-entry issues only; CLI behaviour unchanged
    - tests/test_submit.py
    - tests/test_submit_workflow.py
    - tests/test_validate.py        # only if the refactor needs it
    - CONTRIBUTING.md
  read_only:
    - src/gallery/registry.py
    - src/gallery/detect.py
    - schema/
    - taxonomy.yml
    - templates/
    - tests/helpers.py
    - pyproject.toml
    - uv.lock
checks:
  - uv sync
  - uv run pytest
  - uv run ruff check .
  - uv run python -m gallery.validate
  - uvx --from actionlint-py actionlint .github/workflows/submit.yml
  - uv run python -m gallery.submit check on three fake event files in the scratch dir
    (a new public repo such as https://github.com/SpecsFirst/.github; the url of an
    entry already listed; a body with an unknown tag and a bad url): show the three
    submission.json and comments in Result
```

No new runtime dependency (PyYAML is already there).

## Output contract

```yaml
artifacts:
  - .github/ISSUE_TEMPLATE/submit-template.yml
  - .github/workflows/submit.yml
  - src/gallery/submit.py
  - tests/test_submit.py
  - tests/test_submit_workflow.py
  - CONTRIBUTING.md
contract:
  form:
    - Issue form "Submit a template", `title: "Template: "`, `labels: [submission]`.
      Fields, in order, with these ids:
        `url`         input, required: "Repository URL" — public https URL, GitHub or
                      GitLab (any public git forge works).
        `name`        input, required: "Name" — 3 to 60 characters; it gives the
                      template's id (shown in its URL).
        `description` input, required: "One-line description" — 20 to 200 characters.
        `tags`        input, required: "Tags" — comma-separated; exactly one kind and up
                      to 4 tags from taxonomy.yml (link to it).
      A short markdown intro says that license, last update, Copier support and the file
      tree are detected, and that a 👍 on the template's Discussion is a star.
    - No language field: it is optional in the schema and defaults to en.
  check (`python -m gallery.submit check --event FILE --root DIR --out DIR`):
    - Reads `issue.number`, `issue.body` and `issue.state` from the event JSON. A closed
      issue gives no output and exit 0 (nothing to do).
    - Parses the body by the `### <label>` headings of the form, labels read from
      `.github/ISSUE_TEMPLATE/submit-template.yml` itself (not repeated in code);
      `_No response_` is empty. Body capped at 64 KiB.
    - url: trimmed, one trailing `/` removed. name and description: trimmed, whitespace
      collapsed. tags: split on commas, trimmed, lowercased, empty items dropped,
      duplicates dropped. Exactly one item must be a kind → `kind`; the others → `tags`,
      a deprecated tag replaced by `Taxonomy.resolve_tag`. Unknown items are errors that
      list the valid kinds and tags.
    - id = slug of the name: NFKD to ASCII, lowercase, runs of other characters → `-`,
      trimmed, at most 50 characters cut at a `-`; must match the registry id pattern.
    - Refused, with a clear message each: an id already in `templates/`; a url already
      used by an entry (compared without trailing `/` and `.git`, case-insensitive host);
      every schema and validator error of the entry (all reported at once); detect status
      `failed` (its reason, from detect's error codes, in words).
    - The entry is written as the existing entries are (`name`, `url`, `description`,
      `kind`, `tags` in this order, `tags` omitted when empty, flow-style list), then
      validated in place with the registry's own checks, then detected with
      `detect_entry` (no token in the environment).
    - Writes `<out>/submission.json` (format of architecture.md) and, when ok,
      `<out>/templates/<id>.yml`. Exit 0 in both cases; non-zero only for an internal
      failure (bad event file, unreadable form).
    - Comment (Markdown): when ok, the entry in a fenced yaml block (fence longer than any
      backtick run inside), then the detected license, last update, Copier yes/no and the
      tree summary line; when not ok, a list of the problems, each a fixed sentence naming
      the field. User values only ever appear inside code spans or the fenced block, with
      `@` and `<` unable to act (code blocks do that); never in a heading or a link.
  workflow (`.github/workflows/submit.yml`):
    - `on: issues: types: [opened, edited]`; jobs run only if the issue has the label
      `submission`. Top-level `permissions: {}`. `concurrency` group per issue number,
      cancel-in-progress true.
    - Job `check`: permissions `contents: read`; checkout of `main` with persist-credentials
      false; setup-uv, setup-python, `uv sync --frozen` as validate.yml; run
      `gallery.submit check --event "$GITHUB_EVENT_PATH"` with no token in env; upload
      the out dir with actions/upload-artifact@v4 (short retention).
    - Job `propose`: `needs: check`, permissions `contents: write`, `pull-requests: write`,
      `issues: write`; checkout of `main`; download the artifact; validate `entry_file`
      against `^templates/[a-z0-9]+(-[a-z0-9]+)*\.yml$` before use; then
        - ok: branch `submission/<N>` (created from main, or the existing one fetched),
          remove the `templates/*.yml` files the branch added earlier, add the new one,
          commit as github-actions[bot] ("Add template <id> (#N)"), push without force;
          `gh pr create --base main --head submission/<N>` with the json's `title` and a
          body = comment + "Closes #N", or `gh pr edit` when the pull request exists;
          comment on the issue with the pull request link;
        - not ok: comment on the issue with the errors, no branch.
      Text goes through `--body-file` / `--title` from files or env, never through `${{ }}`
      inside `run:`. `GH_TOKEN: ${{ github.token }}` only on the steps that call gh or push.
  contributing:
    - CONTRIBUTING.md leads with the form (link
      `https://github.com/SpecsFirst/gallery/issues/new?template=submit-template.yml`): what
      to fill, what happens next (pull request, review, merge, Discussion, stars). The manual
      pull request stays as a short second section with the fields table. The giscus
      paragraph becomes stars on the Discussion (👍, GitHub account, refreshed every 6 h).
      Detected list: license, `copier.yml`, last update, file tree (no preview, no tools).
  tests:
    - Parser: a real form body (copy GitHub's format: `### Label\n\nvalue`), `_No response_`,
      extra or missing headings, CRLF, body over the cap.
    - Entry: kind/tags split (no kind, two kinds, unknown, deprecated, > 4 tags), slug cases
      (accents, punctuation, only symbols, 60-char name), duplicate id, duplicate url
      variants, schema errors all reported, detect failure (detect stubbed), YAML output
      byte for byte.
    - Comment: hostile values (`@user`, `<img>`, backticks, `](javascript:`, `::error::`)
      stay inert.
    - Workflow: parsed YAML asserts triggers, label condition, permissions per job, token
      only where listed, no `${{` inside any `run:`, persist-credentials false in check,
      the entry_file pattern check, no `--force`.
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
