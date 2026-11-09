---
id: TASK-009
title: Submit a template through an issue form
status: done             # open | in_progress | blocked | done | needs_review | rejected
parent: subrepos/gallery/main.md
subrepo: gallery
issue:
assignee: coder
model: anthropic/claude-sonnet-5-5
created: 2026-11-09
updated: 2026-11-09
---

# TASK-009: Submit a template through an issue form

> One task = one goal. This file is the **exchange contract** between the
> orchestrator (writer) and the atomic coder (reader/executor).

## Goal

Let anyone submit a template by filling a GitHub issue form: an Action checks the repository
and comments on the issue with the entry and a one-click link to propose
`templates/<id>.yml` from the author's fork, or with clear errors. The bot never writes to the
repository. Update `CONTRIBUTING.md` to lead with the form.

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
  - the propose link of the first case, opened by the orchestrator in a browser logged in
    to GitHub, shows the prefilled file (not checkable by the coder)
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
    - Writes `<out>/submission.json` (format of architecture.md). Exit 0 whether the
      submission is ok or not; non-zero only for an internal failure (bad event file,
      unreadable form).
    - Comment (Markdown, at most 60 000 characters): a hidden marker
      `<!-- gallery-submission -->` first; when ok, the entry in a fenced yaml block (fence
      longer than any backtick run inside), the detected license, last update, Copier
      yes/no and the tree summary line, then the steps: 1. open the propose link,
      2. "Commit changes…" then "Propose changes", 3. in the pull request description keep
      or add `Closes #N`. When not ok, a list of the problems, each a fixed sentence naming
      the field, and "edit this issue to fix them: the check runs again". User values only
      ever appear inside code spans or the fenced block; never in a heading or link text.
    - Propose link: `https://github.com/<repo>/new/main?filename=templates/<id>.yml&value=`
      + the entry YAML, both percent-encoded with `urllib.parse.quote(..., safe="")` for the
      value; `<repo>` from `GITHUB_REPOSITORY`, default `SpecsFirst/gallery`. Only our own
      id (pattern-checked) and our own YAML go in it.
  workflow (`.github/workflows/submit.yml`):
    - `on: issues: types: [opened, edited]`; jobs run only if the issue has the label
      `submission`. Top-level `permissions: {}`. `concurrency` group per issue number,
      cancel-in-progress true.
    - Job `check`: permissions `contents: read`; checkout of `main` with persist-credentials
      false; setup-uv, setup-python, `uv sync --frozen` as validate.yml; run
      `gallery.submit check --event "$GITHUB_EVENT_PATH"` with no token in env; upload
      the out dir with actions/upload-artifact@v4 (short retention).
    - Job `comment`: `needs: check`, permissions exactly `issues: write`, no checkout, runs
      no repository code. Downloads the artifact, takes the issue number from the event
      file (not from the artifact), and posts the comment from `submission.json` with
      `gh issue comment <N> --repo "$GITHUB_REPOSITORY" --body-file <file>`. On `edited`,
      it updates the bot's previous comment (the one with the marker, found with `gh api`)
      instead of adding one. `GH_TOKEN: ${{ github.token }}` on that step only.
    - No job has `contents: write` or `pull-requests: write`; nothing pushes, nothing opens
      a pull request. Text goes through files or env, never through `${{ }}` inside `run:`.
  contributing:
    - CONTRIBUTING.md leads with the form (link
      `https://github.com/SpecsFirst/gallery/issues/new?template=submit-template.yml`): what
      to fill, what happens next (bot comment, propose link, your pull request with
      `Closes #N`, checks, merge by a maintainer, Discussion, stars). The manual
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
      stay inert; marker present; size cap; propose link decodes back to the exact
      filename and YAML.
    - Workflow: parsed YAML asserts triggers, label condition, permissions per job, token
      only on the comment step, no `${{` inside any `run:`, persist-credentials false in
      check, no checkout in comment, no `contents: write` / `pull-requests: write` anywhere,
      no `git push` and no `gh pr` in the file.
returns: the branch with its commit(s), and the filled Result section
```

## Attempts

> Filled in by the orchestrator, one line per (re)launch.

- 2026-11-09 #1 (Claude Sonnet subagent, parallel to TASK-006): done. Review re-ran pytest (412
  passed), ruff, validate and actionlint, read the workflow (no write but `issues: write` on one
  step, no `${{` in scripts, comment found only among the bot's own) and the escaping. The
  browser check of the propose link is left to the operator (no browser here).
- 2026-11-09: the operator opened the propose link: file name and content prefilled. Merged as
  gallery#8.

## Result

> Filled in by the executor, **always**, even on failure: set `status` to
> `done` or `blocked` before exiting. Do not delete the contract above.

### Outcome

Done. Three commits on `task/009-submit-form` (not pushed):

- `7f48bd5` refactor(validate): expose the issues of one entry
- `bec456b` feat(submit): check a template submitted through an issue form
- `b98a0ac` docs: lead CONTRIBUTING with the submission form

Checks (all in the worktree):

- `uv sync`: ok.
- `uv run pytest`: 412 passed (about 137 new, in `tests/test_submit.py`, `tests/test_submit_workflow.py`, two in `tests/test_validate.py`).
- `uv run ruff check .`: all checks passed.
- `uv run python -m gallery.validate`: exit 0.
- `uvx --from actionlint-py actionlint .github/workflows/submit.yml`: no finding, exit 0.
- Operator check, not done: open the propose link of case 1 in a browser logged in to GitHub and see the prefilled file.
- `python -m gallery.submit check` on three fake events (real run, network, detect not stubbed; the events and outputs are in `tasks/.exchange/tmp/task-009/`):


**1. New public repo (https://github.com/SpecsFirst/.github), CRLF body**

```json
{"issue": 11, "ok": true, "id": "specsfirst-community-files"}
```

Comment:

> <!-- gallery-submission -->
> ## Your template is ready to propose
> 
> The checks pass. This is the entry that will be added:
> 
> ```yaml
> name: SpecsFirst community files
> url: https://github.com/SpecsFirst/.github
> description: Default community health files of the SpecsFirst organization, for everyone.
> kind: software
> tags: [multi-repo]
> ```
> 
> Detected from the repository:
> 
> - License: `MIT`
> - Last update: `2026-11-06T14:20:00Z`
> - Copier support: no
> - Files: `1 directory, 2 files`
> 
> To propose it:
> 
> 1. [Open the propose link](https://github.com/SpecsFirst/gallery/new/main?filename=templates/specsfirst-community-files.yml&value=name%3A%20SpecsFirst%20community%20files%0Aurl%3A%20https%3A%2F%2Fgithub.com%2FSpecsFirst%2F.github%0Adescription%3A%20Default%20community%20health%20files%20of%20the%20SpecsFirst%20organization%2C%20for%20everyone.%0Akind%3A%20software%0Atags%3A%20%5Bmulti-repo%5D%0A).
> 2. Click **Commit changes…**, then **Propose changes**.
> 3. In the pull request description, keep or add `Closes #11`.
> 
> A maintainer reviews and merges the pull request.


**2. Url of a listed entry (with trailing slash)**

```json
{"issue": 12, "ok": false, "id": null}
```

Comment:

> <!-- gallery-submission -->
> ## The template cannot be proposed yet
> 
> - **Repository URL**: this repository is already listed as `research-thesis`.
> 
> Edit this issue to fix them: the check runs again.


**3. Unknown tag, `@octocat`, two kinds, bad url, short description**

```json
{"issue": 13, "ok": false, "id": null}
```

Comment:

> <!-- gallery-submission -->
> ## The template cannot be proposed yet
> 
> - **Tags**: not in the taxonomy: `bogus`, `@octocat`. Valid kinds: `research`, `software`, `data`. Valid tags: `thesis`, `teaching`, `multi-repo`, `bench`, `sensitive-data`, `open-science`, `security`.
> - **Tags**: use exactly one kind, not 2 (`research`, `software`).
> - **Repository URL**: must be a public address starting with `https://`, without spaces.
> - **One-line description**: must have 20 to 200 characters.
> 
> Edit this issue to fix them: the check runs again.


Propose link of case 1 (for the operator's browser check):

`https://github.com/SpecsFirst/gallery/new/main?filename=templates/specsfirst-community-files.yml&value=name%3A%20SpecsFirst%20community%20files%0Aurl%3A%20https%3A%2F%2Fgithub.com%2FSpecsFirst%2F.github%0Adescription%3A%20Default%20community%20health%20files%20of%20the%20SpecsFirst%20organization%2C%20for%20everyone.%0Akind%3A%20software%0Atags%3A%20%5Bmulti-repo%5D%0A`

Design notes:

- `validate.entry_issues(entry, path, taxonomy, schema)` returns the issues of one entry. `Issue` got two optional fields, `field` and `code`, so `submit.py` words each problem as a fixed sentence and never reuses a validator message (those quote user values). The CLI output is unchanged.
- The entry is written to a temp dir as `templates/<id>.yml`, read back with `load_entry`, compared with what was built, then validated: a YAML mismatch is an internal failure (exit 1), never a wrong entry.
- A body over 64 KiB is refused with a problem (not truncated). A closed issue prints nothing to stdout and writes no `submission.json`; `check` then outputs `found` unset and the `comment` job is skipped.
- The log prints only our own words (`submit: issue #N: ok|K problem(s)`), no issue content.
- Workflow: `check` outputs `found`; `comment` runs only when it is `true`. `comment` validates the artifact with `jq -e` (string, starts with the marker, at most 60 000 characters) before the token step, takes the issue number from the event file, and finds the bot's earlier comment among those authored by `github-actions[bot]` that start with the marker.
- Tests parse the hostile comments with markdown-it-py (already a dependency) and assert no raw HTML (except the marker), image, foreign link or mention appears outside code.

### Artifacts produced

- `.github/ISSUE_TEMPLATE/submit-template.yml`
- `.github/workflows/submit.yml`
- `src/gallery/submit.py`
- `src/gallery/validate.py` (adds `entry_issues`, `Issue.field`, `Issue.code`)
- `tests/test_submit.py`, `tests/test_submit_workflow.py`, `tests/test_validate.py` (2 tests)
- `CONTRIBUTING.md`
- Scratch: `tasks/.exchange/tmp/task-009/` (3 events, 3 outputs)

### Opinion / concerns

- A user who is not the author of the repository can submit it; nothing proves ownership. The maintainer review of the pull request is the control. Fine for a gallery, worth knowing.
- The bot can be made to fetch any public https URL (bounded by detect's timeout and size limits, no token in the environment). Detect treats it as hostile (ADR-002), so I see no new exposure.
- `edited` fires on title edits too, so each edit re-runs a clone; concurrency with cancel-in-progress bounds that to one run per issue.
- Operator setup 6 (label `submission`) must exist for the form to apply it; `submit.yml` does nothing without it. It is marked done in `main.md`.
- The propose link for `GITHUB_REPOSITORY` defaults to `SpecsFirst/gallery`; a fork running the workflow gets its own repository, which is intended.
- `site/index.html` still says "A pull request with one small file, see CONTRIBUTING.md." (TASK-006/007 own `site/`): worth updating to mention the form once those merge.
- `ruff format` was not applied to old files (the repo is not formatted with it); only new files were formatted.

### CLARIFICATION NEEDED

None.
