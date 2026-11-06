---
title: <short audit title>
date: <YYYY-MM-DD>
status: draft            # draft | final
scope: [<subrepo>]       # sub-repos covered
audited:                 # one <namespace>/<project>@<short sha> per repo read
  - <namespace>/<project>@<sha>
requested_by: <operator>
Co-authored-by: <model id> via <provider> <address>   # agent audit only
Assisted-by: <agents-kit URL>/-/commit/<sha> (<tool> <version>)   # agent audit only
---

# <short audit title> — <YYYY-MM-DD>

> An audit is a **dated snapshot** of the revisions listed in `audited`.
> Rules (naming, identity, finding references): `../conventions.md` § Audits.

## Question

What was asked, and against which goal (link `../main.md` or a sub-repo spec).

## Verdict

The answer, in a few sentences. A reader who stops here has the essentials.

## Method

What was read, run and measured (commands, environment, reproductions).

## Measurements

| Check | Result |
|-------|--------|
| {tests / lint / run} | {figures} |

## Strengths (keep)

1. {what is right and must not be lost}

## Findings

One bullet per finding, grouped by prefix. Keep tables for short values only:
long text in table cells is unreadable in a narrow preview.

- **B1 · {short title}.** {what is wrong}
  - *Evidence:* {how it was observed}
  - *Fix:* {suggested fix}

## Gaps vs goal

What is missing to reach the goal, as opposed to what is broken.

## Recommendations

Ordered by priority, each tied to findings or gaps.

## Open questions

Decisions that belong to the owner. Copy them into `../main.md`.

## Follow-up

Where each finding went (filled in when it is turned into work).

| Ref. | Destination |
|------|-------------|
| B1 | `subrepos/{name}/tasks/` backlog / TASK-{NNN} / ADR-{NNN} |
