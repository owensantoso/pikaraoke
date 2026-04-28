---
type: architecture-overview
title: Architecture
domain: architecture
status: active
created_at: "2026-04-29 03:35:25 +09:00"
updated_at: "2026-04-29 03:35:25 +09:00"
repo_state:
  based_on_commit: c406a10
  last_reviewed_commit: c406a10
---

# Architecture

This is a current-state architecture map for agent work. It describes what is on
disk now, not a proposed rewrite.

## System Shape

PiKaraoke is a monolithic Python package with clear internal manager boundaries.
Flask routes expose browser and API surfaces, while a central `Karaoke` object
wires runtime managers together and bridges events to Socket.IO updates.

```text
CLI
  -> pikaraoke.app:main
    -> Flask / Socket.IO / Babel / API blueprints
    -> Karaoke coordinator
      -> PreferenceManager
      -> KaraokeDatabase
      -> SongManager + LibraryScanner
      -> QueueManager
      -> DownloadManager
      -> PlaybackController
        -> StreamManager
          -> FileResolver + FFmpeg helpers
```

## Main Boundaries

| Boundary | Owns | Key files |
|---|---|---|
| `AREA-RUNTIME` | Flask app, blueprints, Socket.IO, server startup, browser splash launch. | `pikaraoke/app.py`, `pikaraoke/lib/args.py`, `pikaraoke/lib/browser.py` |
| Runtime coordination | Manager construction, event wiring, QR code, main playback loop. | `pikaraoke/karaoke.py` |
| `AREA-PLAYBACK` | Queue state, fairness, playback state, FFmpeg streaming, HTTP stream serving. | `pikaraoke/lib/queue_manager.py`, `pikaraoke/lib/playback_controller.py`, `pikaraoke/lib/stream_manager.py`, `pikaraoke/routes/stream.py` |
| `AREA-LIBRARY` | Song list, database sync, file delete/rename, metadata extraction, yt-dlp downloads. | `pikaraoke/lib/song_manager.py`, `pikaraoke/lib/library_scanner.py`, `pikaraoke/lib/karaoke_database.py`, `pikaraoke/lib/download_manager.py`, `pikaraoke/lib/youtube_dl.py` |
| Preferences | `config.ini` persistence, defaults, CLI override hydration. | `pikaraoke/lib/preference_manager.py` |
| `AREA-WEB-UI` | Server-rendered pages, routes, static JS/CSS, Socket.IO client behavior. | `pikaraoke/routes/`, `pikaraoke/templates/`, `pikaraoke/static/` |

## Architecture Areas

Area docs are the stable boundary contracts for subagent work:

- [AREA-RUNTIME](../architecture/areas/AREA-RUNTIME.md) - startup, CLI, server composition, browser launch.
- [AREA-LIBRARY](../architecture/areas/AREA-LIBRARY.md) - song catalog, scanning, metadata, downloads.
- [AREA-PLAYBACK](../architecture/areas/AREA-PLAYBACK.md) - queue, now-playing state, FFmpeg, streaming.
- [AREA-WEB-UI](../architecture/areas/AREA-WEB-UI.md) - routes, templates, static assets, admin/control UI.

## Event Model

`EventSystem` is the internal pub/sub seam. Managers emit events such as
notifications, queue updates, playback starts, song ends, sync lifecycle, and
download lifecycle. `Karaoke` and `app.py` bridge selected events to Socket.IO.

This keeps most manager code from depending directly on Flask or Socket.IO.

## Persistence Model

- Preferences persist to `config.ini` through `PreferenceManager`.
- Song library state is stored in SQLite through `KaraokeDatabase`, defaulting
  to `pikaraoke.db` in the platform data directory.
- Song media files live under the configured download path.
- Temporary stream/transcode output is resolved and cleaned through
  `FileResolver` helpers.

## Critical Flows

### Startup

1. `pikaraoke.app` parses CLI args and creates global Flask/Socket.IO/Babel/API
   objects.
2. `main()` compiles translations, checks FFmpeg, warns about missing JavaScript
   runtime, and creates the download path.
3. `Karaoke` initializes preferences, platform info, database, scanner, song
   manager, playback controller, queue manager, and download manager.
4. `app.config["KARAOKE_INSTANCE"]` exposes the runtime to routes.
5. The gevent WSGI server starts and optional headed browser splash launches.

### Library Sync

1. Startup warms songs from SQLite when possible.
2. A background scan reconciles disk paths with DB paths.
3. `LibraryScanner` detects adds, deletes, moves, and too-many-missing circuit
   breaker cases.
4. `Karaoke._apply_scan_result()` refreshes `SongManager.songs` and emits
   notifications/socket sync events.

### Download And Queue

1. `routes/search.py` queues a download request.
2. `DownloadManager` serializes downloads through one worker thread.
3. yt-dlp writes files as `%(title)s---%(id)s.%(ext)s`.
4. Successful downloads emit `song_downloaded`; `SongManager` registers the new
   file in `SongList` and SQLite.
5. If requested, the downloaded song enters `QueueManager`.

### Playback And Streaming

1. `Karaoke.run()` waits for a queued song and no active playback.
2. `QueueManager.pop_next()` removes the next item.
3. `PlaybackController.play_file()` asks `StreamManager` to prepare a stream.
4. `FileResolver` resolves CDG/ZIP/ASS and creates unique temp stream names.
5. Stream routes serve HLS or MP4 and mark the song started only if stream ID
   matches the current now-playing URL.
6. End/skip/timeout resets now-playing state, kills FFmpeg, and cleans temp
   files.

## External Dependencies

- FFmpeg is required for playback/transcoding.
- yt-dlp handles YouTube search/download behavior.
- A JavaScript runtime is recommended/needed for some yt-dlp extractor behavior.
- Browser automation/launch behavior is isolated in `Browser`.

## Architecture Rules For Future Work

- Keep route handlers thin; move business behavior into managers.
- Use `PreferenceManager.DEFAULTS` as the single source for user preferences.
- Prefer `EventSystem` for UI notifications and cross-manager update signals.
- Mock subprocess, browser, network, and file-heavy behavior in unit tests.
- Do not introduce a new abstraction unless it replaces repeated complexity or
  matches an existing manager boundary.
