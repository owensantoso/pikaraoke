---
type: plan
id: PLAN-0001
title: Agent Docs Initialization
domain: repo-health
status: draft
created_at: "2026-04-29 03:35:25 +09:00"
updated_at: "2026-04-29 03:35:25 +09:00"
branch: codex/agent-docs-init
related_specs: []
related_adrs: []
related_sessions:
  - AGENT-DOCS/repo-health/session-logs/2026-04-29-agent-docs-initialization.md
linked_paths:
  - AGENTS.md
  - AGENT-DOCS/README.md
  - AGENT-DOCS/orientation/CURRENT_STATE.md
areas:
  - AREA-RUNTIME
  - AREA-LIBRARY
  - AREA-PLAYBACK
  - AREA-WEB-UI
repo_state:
  based_on_commit: c406a10
  last_reviewed_commit: c406a10
---

# PLAN-0001 - Agent Docs Initialization

**Goal:** Give this PiKaraoke fork a growing-profile AGENT-DOCS setup that helps
future agents orient, split work, hand off context, and preserve verification
without accidentally polluting upstream PiKaraoke pull requests.

**Source Spec:** Current working agreement from this session:

- use an official AGENT-DOCS profile name, not an invented size label
- prefer the AGENT-DOCS installer/init flow where possible
- install `docs-meta` because deterministic ID/check/view tooling is central to
  the workflow
- keep the docs fork-local unless explicitly proposed upstream
- preserve a cherry-pick-friendly workflow for future upstream PRs
- make future subagent work easier through durable repo orientation

## Architecture

Keep a fork-local `AGENT-DOCS/` tree at repo root and keep public upstream-facing
docs in the existing `docs/` folder. Root `AGENTS.md` is the first instruction
surface and points agents into `AGENT-DOCS/`.

The official AGENT-DOCS installer writes profile docs under `docs/`, but this
fork intentionally uses `AGENT-DOCS/` to make agent-only files visually separate
from public project docs. Treat that as a local adaptation of the `growing`
profile, not a new profile.

## Prerequisites

- `origin` points at `owensantoso/pikaraoke`.
- `upstream` points at `vicwomg/pikaraoke`.
- Work stays on `codex/agent-docs-init`.
- Agent-doc changes stay in their own commits.

## Locked Decisions

- Profile: `growing`.
- Docs-meta: installed under `scripts/docs-meta` and run with `--root AGENT-DOCS`.
- Agent docs location: `AGENT-DOCS/`.
- Upstream PR hygiene: create clean branches from `upstream/master` and
  cherry-pick only non-agent-doc commits unless the user explicitly wants these
  docs proposed upstream.

## Out Of Scope

- Installing the full AGENT-DOCS scaffold.
- Moving PiKaraoke's public `docs/README.md`.
- Opening an upstream PR for these docs.
- Refactoring PiKaraoke code.

## Task Dependencies / Parallelization

Most work is sequential because this is a small documentation setup and the
root rules should guide every other doc.

Safe parallel work later:

- one agent can refine architecture/current-state docs
- one agent can audit testing/codebase-map accuracy
- one agent can draft future specs/plans for real PiKaraoke changes

Do not run parallel edits on the same docs unless ownership is split by file.

## Implementation Tasks

### Task 1: Normalize Profile Language

**Goal:** Make profile terminology match AGENT-DOCS.

- Replace any informal "medium" language with `growing`.
- Record why `growing` fits PiKaraoke.

Dependencies / parallelization:

- Depends on: prerequisites only.
- Can run in parallel with: none.

### Task 2: Preserve Fork And PR Hygiene Rules

**Goal:** Keep future upstream PRs cherry-pick-friendly.

- Keep root `AGENTS.md` explicit about separate commits and cherry-picking.
- Ensure `AGENT-DOCS/README.md` repeats the upstream PR reminder.

Dependencies / parallelization:

- Depends on: Task 1.
- Can run in parallel with: Task 3.

### Task 3: Reconcile With The Official Installer

**Goal:** Explain the local adaptation clearly enough that future agents do not
mistake it for a new profile.

- Keep a note that `agent-docs-init --profile growing --docs-meta no` was
  dry-run for comparison.
- Record why this fork uses `AGENT-DOCS/` instead of installer-default `docs/`.

Dependencies / parallelization:

- Depends on: Task 1.
- Can run in parallel with: Task 2.

### Task 4: Finish The Growing-Profile Starter Docs

**Goal:** Give future agents enough context to work without chat history.

- Current state.
- Onboarding.
- Architecture.
- Codebase map.
- Testing guide.
- Session log conventions and initial receipt.
- Generated views.

Dependencies / parallelization:

- Depends on: Tasks 2 and 3.
- Can run in parallel with: none unless split by file ownership.

### Task 5: Validate Docs Hygiene

**Goal:** Confirm the docs are internally consistent and easy to commit.

- Run `python scripts/docs-meta --root AGENT-DOCS update`.
- Run `python scripts/docs-meta --root AGENT-DOCS check`.
- Run `python scripts/docs-meta --root AGENT-DOCS check-links`.
- Run `python scripts/docs-meta --root AGENT-DOCS check-todos`.
- Run `git diff --check`.
- Search for stale profile terms like `medium`.
- Confirm no agent-doc files are mixed with code/test changes.

Dependencies / parallelization:

- Depends on: Task 4.
- Can run in parallel with: none.

### Task 6: Commit As Agent-Docs-Only Work

**Goal:** Preserve upstream PR hygiene.

- Commit only `AGENTS.md` and `AGENT-DOCS/`.
- Include `scripts/docs-meta` and `tests/docs-meta-smoke.sh` as agent-docs tooling.
- Do not include unrelated PiKaraoke code or generated dependency changes.

Dependencies / parallelization:

- Depends on: Task 5.
- Can run in parallel with: none.

## Implementation Briefs Needed

No separate `IMPL-*` implementation briefs are needed yet. If this grows into
multiple independent docs audits, create briefs by file group:

- orientation docs
- repo-health docs
- future specs/plans/ADRs

## Validation

Required before claiming the docs init is ready:

```powershell
git diff --check
python .\scripts\docs-meta --root AGENT-DOCS update
python .\scripts\docs-meta --root AGENT-DOCS check
python .\scripts\docs-meta --root AGENT-DOCS check-links
python .\scripts\docs-meta --root AGENT-DOCS check-todos
Select-String -Path .\AGENTS.md,.\AGENT-DOCS\**\*.md -Pattern 'medium|Medium'
git status --short --branch
```

Optional:

```powershell
uv run pytest tests/ -q
```

The optional test suite is not required for docs-only changes, but the existing
baseline was `702 passed, 1 skipped` before the docs initialization.

## Completion Criteria

- Future agents can start at `AGENTS.md` and find the correct read order.
- Future agents can use `docs-meta` for IDs, views, checks, links, todos, and
  health.
- Future agents can use `AREA-*` docs to split runtime, library, playback, and
  web UI work safely.
- The docs clearly say `growing`, not `medium`.
- The plan records the local adaptation away from installer-default `docs/`.
- A future upstream PR can exclude these docs by cherry-picking only non-agent
  commits onto a clean branch from `upstream/master`.
- The working tree contains only the intended agent-docs changes before commit.
