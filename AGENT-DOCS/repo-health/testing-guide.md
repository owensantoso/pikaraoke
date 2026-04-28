---
type: testing-guide
title: Testing Guide
domain: repo-health
status: active
created_at: "2026-04-29 03:35:25 +09:00"
updated_at: "2026-04-29 03:35:25 +09:00"
repo_state:
  based_on_commit: c406a10
  last_reviewed_commit: c406a10
---

# Testing Guide

Use this to choose validation for agent work.

## Baseline

The local unit suite passed on this fork before agent-doc initialization:

```powershell
uv run pytest tests/ -q
```

Result:

```text
702 passed, 1 skipped
```

`uv run` created a local `.venv`; it is ignored by git.

## Normal Validation

For most code changes:

```powershell
uv run pytest tests/ -q
```

For targeted changes, run the narrow test first, then the full suite when the
blast radius is not trivial:

```powershell
uv run pytest tests/unit/test_queue_manager.py -q
uv run pytest tests/ -q
```

For style and repo quality checks:

```powershell
uv run pre-commit run --config code_quality/.pre-commit-config.yaml --all-files
```

For agent docs and docs metadata changes:

```powershell
python scripts/docs-meta --root AGENT-DOCS update
python scripts/docs-meta --root AGENT-DOCS check
python scripts/docs-meta --root AGENT-DOCS check-links
python scripts/docs-meta --root AGENT-DOCS check-todos
bash tests/docs-meta-smoke.sh
```

On this Windows setup, Bash may print a WSL drive-mount warning even when the
smoke test passes. Treat the script's exit code and `docs-meta smoke test
passed` output as the result.

## CI Expectations

`.github/workflows/ci.yml` runs:

- code quality via pre-commit
- docstring coverage
- commitlint
- multi-OS smoke tests
- unit tests with coverage
- Docker smoke tests for amd64 and arm64

## Test Design Notes

- Mock external I/O and subprocess operations.
- Prefer real `EventSystem` and `PreferenceManager` instances because they are
  lightweight and match upstream guidance.
- Put manager behavior tests close to the relevant manager module.
- For route changes, inspect existing route tests before inventing new fixture
  patterns.
- Avoid live YouTube, FFmpeg, browser, network, or platform dependencies in unit
  tests unless the user explicitly asks for an integration/smoke test.

## Focused Test Commands By Area

```powershell
# Queue and playback coordination
uv run pytest tests/unit/test_queue_manager.py tests/unit/test_queue_routes.py tests/unit/test_playback_controller.py -q

# Downloads and YouTube integration seams
uv run pytest tests/unit/test_download_manager.py tests/unit/test_youtube_dl.py -q

# Library, database, and file resolution
uv run pytest tests/unit/test_library_scanner.py tests/unit/test_karaoke_database.py tests/unit/test_file_resolver.py -q

# Preferences and routes
uv run pytest tests/unit/test_preference_manager.py tests/unit/test_preference_routes.py -q

# Metadata/title cleanup and batch renaming
uv run pytest tests/unit/test_metadata_parser.py tests/unit/test_batch_renamer.py -q
```

Use the focused command while iterating, then run `uv run pytest tests/ -q` when
the change crosses manager/route boundaries.

## Completion Rule

Do not say work is complete unless the relevant validation has been run, or the
reason it could not be run is recorded in the final response and session log.
