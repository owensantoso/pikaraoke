# Agent Instructions

This repository is a fork of `vicwomg/pikaraoke`. Treat upstream contribution
hygiene as a first-class rule.

## Fork And PR Hygiene

The agent docs workflow may live in this fork even if upstream does not want it.
Do not assume agent docs belong in pull requests to upstream PiKaraoke.

- Keep agent-docs changes and upstreamable PiKaraoke changes in separate commits.
- Do not mix `AGENTS.md`, `AGENT-DOCS/`, docs-meta scaffolding, session logs, or
  agent-only planning files into the same commit as product/code/test changes.
- When preparing an upstream PR, create a clean branch from `upstream/master` and
  cherry-pick only the non-agent-doc commits.
- If a change intentionally updates agent docs and product code together, split it
  before pushing or opening a PR unless the user explicitly says the docs should
  be proposed upstream.

Example clean PR flow:

```powershell
git fetch upstream
git switch -c feature/my-change upstream/master
git cherry-pick <code-change-commit> <test-change-commit>
git push -u origin feature/my-change
```

## Repo Basics

- Python package entry point: `pikaraoke.app:main`.
- Main coordinator: `pikaraoke/karaoke.py`.
- Focused managers live under `pikaraoke/lib/`.
- Flask routes live under `pikaraoke/routes/`.
- Tests live under `tests/unit/`.

## Agent Docs

Fork-local agent workflow docs live under `AGENT-DOCS/`.

- Start with `AGENT-DOCS/README.md`.
- Treat `AGENT-DOCS/orientation/CURRENT_STATE.md` as the quick current-state map.
- Use `AGENT-DOCS/repo-health/codebase-map.md` before delegating broad codebase work.
- Record durable session receipts in `AGENT-DOCS/repo-health/session-logs/`.
- Use `python scripts/docs-meta --root AGENT-DOCS ...` for stable IDs, generated
  views, checks, links, todos, and docs health.
- Keep these files in agent-docs commits unless the user explicitly wants to propose them upstream.

## Validation

Use the repo's normal validation for code changes:

```powershell
uv run pytest tests/ -q
```

For broader quality checks, use:

```powershell
uv run pre-commit run --config code_quality/.pre-commit-config.yaml --all-files
```

Do not claim a change is ready until the relevant validation has been run or the
reason it could not be run is recorded.
