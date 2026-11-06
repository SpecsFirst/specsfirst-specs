# Conventions

> Shared conventions so all tasks and agents stay consistent across the
> project.
>
> **Forge:** every repo of this project is on GitHub (org `SpecsFirst`), so read
> *pull request* for *merge request*, and use the GitHub forms below (`/blob/`,
> `/commit/`, `gh pr create`). The specs and agents-kit URLs in trailers are built
> by the scripts.

## Naming

- Task files: `TASK-<NNN>-<slug>.md` (zero-padded, hyphenated slug).
- Branches: `feature/<slug>`, `fix/<slug>`, `task/<NNN>-<slug>`.
- Worktrees: `<project>/.worktrees/<repo>-<slug>` (named by intent).
- Commits: follow the target repo's existing style (default: conventional
  `feat:`, `fix:`, `docs:`, `chore:`).
- Agent commits end with **3 trailers, plus `Refs` when the task has an
  issue**, stamped by the launcher, never typed by the model:

  ```
  Refs: #42
  Task: https://<forge>/<namespace>/<project>-specs/-/blob/<short sha>/subrepos/<subrepo>/tasks/TASK-<NNN>-<slug>.md
  Co-authored-by: <exact model id> via <provider> <address>
  Assisted-by: https://<forge>/<namespace>/agents-kit/-/commit/<short sha> (<tool> <version>)
  ```

  | Trailer | Value | Always? |
  |---------|-------|---------|
  | `Refs` | the task's `issue:` (`#N` same project, `group/project#N` otherwise) | only with an issue |
  | `Task` | URL of the spec file at `<short sha>` (`<specs repo>/-/blob/<sha>/subrepos/<subrepo>/tasks/TASK-<NNN>-<slug>.md`) — the prompt, at the commit that delegated it | coder commits (none on the orchestrator's own spec commits) |
  | `Co-authored-by` | `<exact model id> via <provider> <address>` | yes |
  | `Assisted-by` | `<agents-kit URL>/-/commit/<short sha> (<tool> <version>)`, `; kit dirty` inside the parentheses when the kit has local changes | yes |

  `Task` and `Assisted-by` are full https URLs, built by the scripts from
  the repos' `origin` (GitLab `/-/blob/`, `/-/commit/`; github.com `/blob/`,
  `/commit/`): clickable on any forge. Not `<namespace>/<project>@<sha>`
  references: GitLab links them only within its own instance, and GitHub
  reads them as GitHub repos (older commits keep that form, still accepted by
  `check`). Address = the vendor's official noreply when it exists, else
  `noreply@<provider>.invalid` (RFC 2606: never registrable). Check:
  `git log --format='%h %(trailers:key=Task,valueonly,separator=)'`.
- Task specs are versioned. The orchestrator commits a spec when delegating
  (`task(<subrepo>): TASK-<NNN> <slug>`) and again after review
  (`task(<subrepo>): TASK-<NNN> <status>`); both messages carry
  `Refs: <group/project>#N` when the spec has an `issue:`, so the issue
  timeline links the spec, the code commits and the MR without any API call.

## Protected branches

`main` is protected on every repo of this project: nobody pushes to it directly, and it
changes only through a pull request merged by the operator. This includes this specs
repo, so the orchestrator commits task specs on a branch `specs/<slug>`, pushes it before
launching a coder, and opens one pull request per coherent batch.

A `Task` trailer points at a commit of that branch. It stays valid only if the pull
request is merged with a **merge commit**: a squash or a rebase would break the link.

## Audits

- File: `audit/<YYYY-MM-DD>-<slug>.md`, from `audit/AUDIT-template.md`.
- An audit is a dated snapshot of the revisions listed in `audited`
  (`<namespace>/<project>@<short sha>`). Once `final`, it is never edited to
  track later changes: write a new audit instead.
- An agent audit carries its identity in the `Co-authored-by` and
  `Assisted-by` fields (remove them on a human audit), with the same values
  as its commit trailers, produced by `agents-kit/scripts/agent-trailers
  trailers`, never typed by the model.
- Finding references use a prefix and a number: **B** bug, **D** debt,
  **S** security, **P** performance, plus project-specific prefixes declared
  here: _none yet_ (`B1`, `D2`…). They are stable: backlog lines, tasks and
  ADRs cite them, and the audit's **Follow-up** table records where each one
  went.

## Process

- One logical change per commit.
- Run linters / typecheck / tests before marking a task done.
- A task that changes runtime behaviour lists the check the **operator** runs
  (e.g. a deploy); coders never deploy.
- Security-relevant decisions go to `architecture.md` as an ADR.
- Never force-push, never rewrite shared history, never commit secrets.

## Merge requests

The agent opens the MR, the operator merges. The orchestrator writes it from
the task spec and its Result (never the coder).

- **Granularity:** the orchestrator picks it so each MR reads in one sitting
  and tells one story. Issue: tasks merged by the orchestrator into
  `issue/<N>-<slug>`, one MR `Closes #N`, or one MR per coherent step
  (`Refs #N`, the last `Closes #N`). No issue: one MR per task. A `sprint` /
  `dev` branch only when the operator asks for it.
- **Title:** `TASK-<NNN>: <task title>`; issue MR `#N: <issue title>` (no
  task: the commit style).
- **Target:** `main`; a stacked MR targets the previous task branch.
- **Merge:** merge commit only (no squash, no rebase from the UI: trailers
  and task history must survive); source branch deleted.
- **Open:** GitLab `git push -o merge_request.create -o merge_request.target=main
  -o merge_request.title=… -o merge_request.description=…` (no token);
  GitHub `gh pr create` (PR token). Push options cannot carry newlines: on
  GitLab the description is sent as single-line HTML (`<h2>`, `<ul>`, `<p>`).
  Trailers go in a `<ul>` list (GitLab drops `<br>`), never in `<pre>`/code:
  GitLab does not link references inside code.
  GitHub: push the branch, then `gh pr create --base <target> --head
  <branch> --title … --body-file <file>`; the body is the Markdown below as
  is, trailers as plain lines (GitHub breaks single newlines), with a blank
  line before `---`. Merge commits only and head-branch deletion are repo
  settings there (no per-PR option).
- **Description:**

  ```markdown
  ## Goal
  <one sentence, from the task>

  ## Changes
  - <what changes>

  ## Checks
  - [x] <contract checks, with result>

  ## After merge (operator)
  <what the operator runs or checks, e.g. `make deploy HOST=<alias> TAGS=<roles>`, or "no change expected">

  ## Notes
  <risks, scope deviations, orchestrator fixes; omit if none>

  ---
  <trailer block of the task: Refs / Task / Co-authored-by / Assisted-by>
  ```

  `Refs: #N` only with an issue; `Closes #N` only on the issue's last MR.

## Definitions

| Term | Meaning |
|------|---------|
| spec | A durable, versioned description of intent or a task. |
| contract | The explicit input/output of a task (see task template). |
| operator | The human owning the infrastructure, secrets and forge accounts. |
| agent | Any LLM-driven process working on the project. |

## Model-agnostic note

These specs must stay **independent of any specific model or agent tooling**.
Model behavior and agent roles live in `agents-kit`; this repository holds only
the *what* and *why*, never the *how with a particular model*.
