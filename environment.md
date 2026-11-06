# Environment

> Where tasks run. Read by every coder before starting.

## Machine

- Linux VM, 16 CPU, 62 GB RAM, plenty of disk.
- Tools available: git, python 3.12, uv, jq, gh.
- Not available: node, npm, a browser, shellcheck. Tools missing locally can be run
  through uv when a Python package ships them, e.g.
  `uv run --with actionlint-py actionlint`.

## Limits

- Writable area: your worktree and your task's spec only.
- Scratch files go in `tasks/.exchange/tmp/`.
- Network: allowed for fetching dependencies and public git repositories.
- No push, no pull request, no change of repository settings: the orchestrator
  and the operator handle those.
- Concurrency: one opencode coder at a time (provider quota); Claude coders can
  run in parallel on other repos.

## Recipes

- Python: `uv sync`, then `uv run <command>`. Everything is in the repo's
  `pyproject.toml`; no global install.
