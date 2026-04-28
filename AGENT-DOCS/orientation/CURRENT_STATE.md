---
type: current-state
title: Current State
domain: orientation
status: active
created_at: "2026-04-29 03:35:25 +09:00"
updated_at: "2026-04-29 03:35:25 +09:00"
repo_state:
  based_on_commit: c406a10
  last_reviewed_commit: c406a10
---

# Current State

PiKaraoke is a cross-platform karaoke server for Raspberry Pi, Windows, macOS,
and Linux. It combines a Flask web interface, Socket.IO updates, yt-dlp
downloads, FFmpeg-based playback/transcoding, a local song library, and a
browser splash/player surface.

## Current Branch Context

- Fork: `owensantoso/pikaraoke`.
- Upstream: `vicwomg/pikaraoke`.
- Agent-docs branch: `codex/agent-docs-init`.
- Local baseline before docs initialization: `702 passed, 1 skipped` via
  `uv run pytest tests/ -q`.

## Repo Size Recommendation

Use a **growing** AGENT-DOCS footprint for this repo.

Reason: the codebase has multiple runtime surfaces, recurring handoff value, and
enough manager classes to benefit from persistent orientation, but it does not
need the full scaffold on day one.

Included now:

- root agent rules
- current-state map
- onboarding walkthrough
- architecture overview
- codebase map
- testing guide
- session log home
- deterministic docs-meta tooling
- generated registry, spec, todo, and roadmap views
- architecture area docs for runtime, library, playback, and web UI
- placeholder homes for future specs, plans, and ADRs

Not included yet:

- marketing, operations, and research trees
- repo-health audit cadence

Add those only when active work needs them.

## Fast Repo Map

| Path | Role |
|---|---|
| `pikaraoke/app.py` | Flask app construction, blueprint registration, Socket.IO setup, server startup, browser splash launch. |
| `pikaraoke/karaoke.py` | Main coordinator for preferences, song library, queue, downloads, playback, QR code, and event wiring. |
| `pikaraoke/lib/` | Focused managers and adapters for downloads, playback, streams, preferences, files, metadata, ffmpeg, yt-dlp, network, and platform behavior. |
| `pikaraoke/routes/` | HTTP/API/UI routes grouped by surface. |
| `pikaraoke/templates/` | Server-rendered UI templates. |
| `pikaraoke/static/` | Browser assets, bundled JS/CSS, fonts, images, sounds, videos. |
| `tests/unit/` | Unit tests with mocked external I/O and subprocess-heavy seams. |
| `build_scripts/` | Installers, Docker, and CI smoke-test helpers. |
| `code_quality/` | Pre-commit, pylint, markdown, and YAML quality configuration. |

## Highest-Value Reading Path

1. `CLAUDE.md`
2. `docs/README.md`
3. `pyproject.toml`
4. `pikaraoke/app.py`
5. `pikaraoke/karaoke.py`
6. `pikaraoke/lib/queue_manager.py`
7. `pikaraoke/lib/playback_controller.py`
8. `pikaraoke/lib/stream_manager.py`
9. `pikaraoke/lib/download_manager.py`
10. `tests/conftest.py`

## Current Source Of Truth

- Public user-facing docs: `docs/README.md`.
- Agent/fork workflow docs: `AGENTS.md` and `AGENT-DOCS/`.
- Agent docs metadata tooling: `scripts/docs-meta --root AGENT-DOCS`.
- Architecture boundaries: `AGENT-DOCS/architecture/areas/`.
- Maintainer guidance: `CLAUDE.md`.
- Package/runtime dependencies: `pyproject.toml` and `uv.lock`.
- CI expectations: `.github/workflows/ci.yml`.

## Do Not Assume

- Do not assume agent docs should be included in upstream PRs.
- Do not assume route handlers own business logic; most behavior belongs in
  managers under `pikaraoke/lib/` or in `Karaoke`.
- Do not assume media, network, browser, FFmpeg, or yt-dlp behavior can be tested
  with live external dependencies. Existing tests generally mock those seams.
