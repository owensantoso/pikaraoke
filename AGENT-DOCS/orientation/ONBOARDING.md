---
type: onboarding
title: Onboarding
domain: orientation
status: active
created_at: "2026-04-29 03:35:25 +09:00"
updated_at: "2026-04-29 03:35:25 +09:00"
repo_state:
  based_on_commit: c406a10
  last_reviewed_commit: c406a10
---

# Onboarding

Use this when an agent needs to understand PiKaraoke without reading every file.

## What The App Does

PiKaraoke runs a local karaoke station. One browser surface acts as the TV/player
screen, while phones or other browsers use the web interface to search, browse,
queue, download, and control songs. Local files and YouTube downloads are both
supported.

## Runtime Story

1. The `pikaraoke` console script enters `pikaraoke.app:main`.
2. `app.py` parses CLI arguments, initializes Flask, Babel, Socket.IO, and
   flask-smorest blueprints.
3. Startup checks verify FFmpeg and warn about missing JavaScript runtimes needed
   by yt-dlp.
4. `Karaoke` is constructed with CLI settings and persisted preferences.
5. `Karaoke` initializes the event system, preference manager, database, song
   manager, library scanner, playback controller, queue manager, and download
   manager.
6. The Flask app stores the `Karaoke` instance in app config for route access.
7. A gevent WSGI server starts on the configured port.
8. In headed mode, a browser splash/player screen is launched.
9. `Karaoke.run()` keeps playback moving through queued songs.

## Main User Flows

### Browse Or Search Songs

- Routes in `pikaraoke/routes/files.py` and `pikaraoke/routes/search.py` serve UI
  and actions.
- Local files come from `SongManager` and `SongList`.
- YouTube search/download behavior flows through `youtube_dl.py` and
  `DownloadManager`.

### Queue Songs

- Routes in `pikaraoke/routes/queue.py` accept enqueue, edit, reorder, random,
  and download-status actions.
- `QueueManager` owns queue state, duplicate checks, user limits, fair queueing,
  reorder operations, and queue update events.

### Play Songs

- `Karaoke.run()` pops songs from `QueueManager`.
- `PlaybackController` owns now-playing state and delegates stream preparation to
  `StreamManager`.
- `StreamManager` uses `FileResolver` and FFmpeg helpers to prepare HLS or MP4
  streams.
- Routes in `pikaraoke/routes/stream.py` serve stream segments/files and mark
  playback state when clients connect.

### Manage Preferences

- `PreferenceManager` is the single source for persisted user settings in
  `config.ini`.
- `pikaraoke/routes/preferences.py` changes and clears preferences.
- CLI overrides can be persisted so the UI reflects startup choices.

## Mental Model

- `app.py` composes the web server.
- `Karaoke` coordinates the app runtime.
- `lib/*Manager.py` classes own focused behavior.
- `routes/*.py` should stay thin and call managers.
- `templates/` and `static/` provide the browser UI.
- Tests should target manager logic and route integration with external seams
  mocked.

## First Tasks For A New Agent

1. Read `AGENTS.md`.
2. Read `AGENT-DOCS/orientation/CURRENT_STATE.md`.
3. Read the relevant architecture area:
   - `AGENT-DOCS/architecture/areas/AREA-RUNTIME.md`
   - `AGENT-DOCS/architecture/areas/AREA-LIBRARY.md`
   - `AGENT-DOCS/architecture/areas/AREA-PLAYBACK.md`
   - `AGENT-DOCS/architecture/areas/AREA-WEB-UI.md`
4. Read the task-relevant manager or route.
5. Check matching tests under `tests/unit/`.
6. Keep commits separated by upstreamability.

## Common Change Paths

### Add Or Change A Preference

1. Start at `PreferenceManager.DEFAULTS`.
2. Check `parse_pikaraoke_args()` for CLI exposure.
3. Check `routes/preferences.py` and templates for web settings.
4. Verify config persistence tests and route tests.

### Change Queue Behavior

1. Start at `QueueManager`.
2. Check `routes/queue.py` for HTTP/API behavior.
3. Check `Karaoke.get_now_playing()` if queue preview changes.
4. Run queue manager, route, reorder, and socket tests.

### Change Playback Or Streaming

1. Start at `PlaybackController` and `StreamManager`.
2. Check `FileResolver` for file type handling.
3. Check `routes/stream.py` for browser serving behavior.
4. Mock FFmpeg and filesystem-heavy behavior in unit tests.

### Change Search Or Downloads

1. Start at `routes/search.py`.
2. Check `youtube_dl.py` for command construction and parsing.
3. Check `DownloadManager` for worker/progress behavior.
4. Avoid live YouTube in unit tests.
