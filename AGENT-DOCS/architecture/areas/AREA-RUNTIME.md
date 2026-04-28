---
type: architecture-area
id: AREA-RUNTIME
title: Runtime And App Composition
domain: architecture
status: active
created_at: "2026-04-29 03:35:25 +09:00"
updated_at: "2026-04-29 03:35:25 +09:00"
owners: []
related_specs: []
related_adrs: []
related_plans:
  - PLAN-0001
related_sessions:
  - AGENT-DOCS/repo-health/session-logs/2026-04-29-agent-docs-initialization.md
related_issues: []
related_prs: []
repo_state:
  based_on_commit: c406a10
  last_reviewed_commit: c406a10
---

# AREA-RUNTIME - Runtime And App Composition

## Owns

- CLI entry through `pikaraoke.app:main`.
- Global Flask, Socket.IO, Babel, and flask-smorest API setup.
- Blueprint registration and Swagger exposure.
- FFmpeg/JavaScript runtime startup checks.
- Gevent WSGI server lifecycle and optional browser splash launch.
- App context storage for the shared `Karaoke` instance.

## Does Not Own

- Song queue behavior; see `AREA-PLAYBACK`.
- Song library persistence and downloads; see `AREA-LIBRARY`.
- Browser template behavior and route UX; see `AREA-WEB-UI`.

## Stable Invariants

- `app.py` does global setup at import time, including parsing args and calling
  `gevent.monkey.patch_all()`. Tests that import it need to account for that.
- `main()` should fail early when FFmpeg is missing.
- Missing JavaScript runtime is a warning because yt-dlp may still run for some
  sources.
- The `Karaoke` instance is retrieved by routes through `current_app` helpers,
  not by constructing new runtime objects in route modules.

## Interfaces / Seams

| Interface | Caller | Owns | Does Not Own |
|---|---|---|---|
| `parse_pikaraoke_args()` | `app.py`, tests | CLI argument defaults, validation, conversion. | Preference persistence. |
| `app.config["KARAOKE_INSTANCE"]` | routes | Shared runtime object lookup. | Runtime behavior inside managers. |
| `SocketIO` events | `Karaoke`, routes, templates | Transport for UI updates. | Business rules that decide when state changes. |
| `Browser` | `app.py` | Opening/closing headed splash screen. | Player UI content. |

## Intended Flow

1. Parse CLI arguments.
2. Initialize Flask/Babel/Socket.IO/API blueprints.
3. Compile translations if needed.
4. Verify runtime prerequisites.
5. Create download path and `Karaoke`.
6. Store `Karaoke` in Flask config.
7. Start WSGI server.
8. Launch browser splash unless headless.
9. Enter `Karaoke.run()`.

## Current Mismatch Notes

- Because `args = parse_pikaraoke_args()` happens at module import time, import
  side effects are real. Prefer focused tests around helpers where possible.
- Windows development may have working Python/uv tests even when FFmpeg or
  browser behavior is not available for runtime smoke testing.

## Related Areas

- `AREA-WEB-UI` - consumes Flask and Socket.IO surfaces.
- `AREA-PLAYBACK` - runtime loop delegates playback into this area.
- `AREA-LIBRARY` - runtime startup initializes the song database and scanner.
