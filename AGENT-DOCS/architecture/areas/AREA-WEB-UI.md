---
type: architecture-area
id: AREA-WEB-UI
title: Web UI, Routes, And Client Events
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

# AREA-WEB-UI - Web UI, Routes, And Client Events

## Owns

- Flask route modules under `pikaraoke/routes/`.
- Server-rendered templates under `pikaraoke/templates/`.
- Static browser assets under `pikaraoke/static/`.
- Admin authentication checks and route-level permission behavior.
- Socket.IO event handlers and browser update events.
- Splash/player page behavior, background media, score phrases, and screensaver
  assets.

## Does Not Own

- Queue policy and playback state; see `AREA-PLAYBACK`.
- Library persistence and download processing; see `AREA-LIBRARY`.
- App/server startup; see `AREA-RUNTIME`.

## Stable Invariants

- Routes obtain runtime state through `get_karaoke_instance()`.
- Admin-protected routes should use `is_admin()` and not rely on hidden UI alone.
- Route schemas use `flask_smorest.Blueprint` plus Marshmallow arguments where
  validation matters.
- Business behavior should stay in managers or `Karaoke`; routes should adapt
  HTTP/UI inputs to those calls.
- Browser clients receive real-time updates through Socket.IO events such as
  `queue_update`, `now_playing`, notifications, and control broadcasts.

## Route Map

| Surface | Route module | Notes |
|---|---|---|
| Home and info | `home.py`, `info.py` | Landing/control summaries and system stats. |
| Queue | `queue.py` | Queue page, enqueue, reorder, edit, random, download status. |
| Search/download | `search.py` | YouTube search, local autocomplete, preview, download request. |
| Browse/files | `files.py` | Browse songs, delete, edit/rename. |
| Player/splash | `splash.py`, `stream.py`, `background_music.py`, `images.py` | TV player, streams, QR/logo/background media. |
| Controls | `controller.py`, `socket_events.py` | Skip/pause/restart/volume/transpose and Socket.IO control events. |
| Admin/preferences | `admin.py`, `preferences.py` | Auth, shutdown/reboot/quit, yt-dlp update, library sync, preferences. |
| Metadata tools | `metadata_api.py`, `batch_song_renamer.py` | Title tidy/suggestions and batch rename workflow. |

## Interfaces / Seams

| Interface | Caller | Owns | Does Not Own |
|---|---|---|---|
| `current_app.py` helpers | routes/templates | App config access, admin cookie checks, event broadcasts. | Manager behavior. |
| Jinja templates | route handlers | Server-rendered UI composition. | Core queue/download/playback rules. |
| Static JS/CSS | browser pages | Client navigation, sockets, score/screensaver effects. | Server-side validation. |
| Marshmallow schemas | routes | Request argument/body validation. | Runtime domain invariants. |

## Intended Flow

1. Browser requests page or action route.
2. Route validates request and admin status if needed.
3. Route calls `Karaoke` or a focused manager.
4. Manager emits internal events or route broadcasts explicit control events.
5. Browser receives refreshed HTML, JSON, or Socket.IO updates.

## Current Mismatch Notes

- Some routes still return raw `json.dumps` instead of `jsonify`; follow local
  patterns unless a task is specifically modernizing responses.
- Admin auth uses a cookie whose value matches the configured admin password.
- Swagger docs are available only when `--enable-swagger` is passed.
- The UI is server-rendered with bundled assets, not a separate frontend build.

## Related Areas

- `AREA-RUNTIME` - registers route blueprints and Socket.IO.
- `AREA-PLAYBACK` - route actions mutate queue/playback state.
- `AREA-LIBRARY` - browse/search/file routes expose library behavior.
