# specsfirst-specs

Specifications of the [SpecsFirst](https://github.com/SpecsFirst) project: one
specs repo driving every repo of the organization. It is built from
[`specs-template`](https://gitlab.inria.fr/dev-studio/specs-template).

Start with [`main.md`](main.md). Contributing a template to the gallery does not
require reading any of this: see the gallery's `CONTRIBUTING.md`.

| Path | Purpose |
|------|---------|
| `main.md` | Whole-project view and table of sub-repos. |
| `architecture.md` | Decisions (ADRs) and the data contracts between components. |
| `conventions.md` | Naming, commit trailers, pull requests, audits. |
| `environment.md` | Where agent tasks run: tools and limits. |
| `subrepos/<name>/main.md` | Spec of one sub-repo. |
| `subrepos/<name>/tasks/` | Task board and `TASK-*.md`, the contract of each task. |

Each sub-repo's code lives in its own repo, cloned next to this one. Transient
exchange goes in `tasks/.exchange/` and is git-ignored.
