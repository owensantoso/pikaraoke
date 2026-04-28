# Agent Docs

Fork-local coordination docs for agents working on this PiKaraoke fork.

This repo fits the AGENT-DOCS **growing** profile: roughly 76 Python files plus
Flask routes, server-rendered templates, static browser assets, installers,
Docker files, CI, and a broad unit suite. The useful initialization size is
therefore growing-style: enough orientation and handoff structure to support
subagents, without copying the full scaffold.

## Start Here

1. [../AGENTS.md](../AGENTS.md) - root rules, especially fork and PR hygiene.
2. [CURRENT_STATE.md](orientation/CURRENT_STATE.md) - current repo state and the fastest map.
3. [ONBOARDING.md](orientation/ONBOARDING.md) - product and runtime walkthrough.
4. [ARCHITECTURE.md](orientation/ARCHITECTURE.md) - system shape and main boundaries.
5. [codebase-map.md](repo-health/codebase-map.md) - task-oriented code map.
6. [testing-guide.md](repo-health/testing-guide.md) - validation commands and test seams.
7. [Architecture areas](architecture/README.md) - stable boundaries for subagent work.
8. [PLAN-0001](repo-health/plans/PLAN-0001-agent-docs-initialization/PLAN-0001-agent-docs-initialization.md) - docs initialization plan.
9. [DOCS-REGISTRY.md](DOCS-REGISTRY.md), [AREAS.md](AREAS.md), [SPECS.md](SPECS.md), [TODOS.md](TODOS.md), and [ROADMAP-VIEW.md](ROADMAP-VIEW.md) - generated views.
10. [session logs](repo-health/session-logs/README.md) - session receipts and handoff notes.

## What Lives Here

| Path | Purpose |
|---|---|
| [orientation/](orientation/) | Stable reader-facing context: current state, onboarding, architecture. |
| [architecture/](architecture/) | Stable area boundaries for runtime, library, playback, and web UI work. |
| [repo-health/](repo-health/) | Codebase maps, testing notes, audits, session logs, and reusable handoff context. |
| [product/specs/](product/specs/) | Future `SPEC-*` requirements docs when work is larger than a quick fix. |
| [product/plans/](product/plans/) | Future `PLAN-*` folders and implementation briefs for delegated work. |
| [decisions/adr/](decisions/adr/) | Durable architecture or workflow decisions. |
| generated views | `DOCS-REGISTRY.md`, `SPECS.md`, `TODOS.md`, `ROADMAP-VIEW.md`; regenerate with `docs-meta`. |

## Docs Meta

This fork installs the deterministic `docs-meta` helper under `scripts/docs-meta`.
Because agent docs live in `AGENT-DOCS/` instead of the public `docs/` folder,
always pass `--root AGENT-DOCS`.

Common commands:

```powershell
python scripts/docs-meta --root AGENT-DOCS update
python scripts/docs-meta --root AGENT-DOCS check
python scripts/docs-meta --root AGENT-DOCS check-links
python scripts/docs-meta --root AGENT-DOCS check-todos
python scripts/docs-meta --root AGENT-DOCS next plan
python scripts/docs-meta --root AGENT-DOCS show PLAN-0001
```

Generated views are caches. Do not edit them by hand; update the source docs and
rerun `docs-meta`.

## Working Rules

- Keep agent docs and upstreamable PiKaraoke changes in separate commits.
- Prefer short, durable docs that help the next agent resume work.
- Update `CURRENT_STATE.md` when the important repo map changes.
- Use `docs-meta` instead of guessing stable IDs, hand-maintaining registries,
  or editing generated views.
- Write a session log after meaningful investigation, planning, implementation,
  or verification work.
- Do not duplicate source-of-truth content from the public `docs/` folder; link
  to it when it already owns the answer.

## Upstream PR Reminder

For pull requests to `vicwomg/pikaraoke`, create a clean branch from
`upstream/master` and cherry-pick only non-agent-doc commits unless these docs
are intentionally being proposed upstream.
