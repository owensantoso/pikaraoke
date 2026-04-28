---
type: codebase-map
title: Codebase Map
domain: repo-health
status: active
created_at: "2026-04-29 03:35:25 +09:00"
updated_at: "2026-04-29 03:35:25 +09:00"
repo_state:
  based_on_commit: c406a10
  last_reviewed_commit: c406a10
---

# Codebase Map

Task-oriented map of the current PiKaraoke source tree.

## Read Order By Task

| Task | Start here | Then read |
|---|---|---|
| App startup or CLI behavior | `pikaraoke/app.py` | `pikaraoke/lib/args.py`, `pikaraoke/lib/browser.py`, `build_scripts/ci/` |
| Queue behavior | `pikaraoke/lib/queue_manager.py` | `pikaraoke/routes/queue.py`, `tests/unit/test_queue_manager.py`, `tests/unit/test_queue_routes.py` |
| Playback behavior | `pikaraoke/lib/playback_controller.py` | `pikaraoke/lib/stream_manager.py`, `pikaraoke/routes/stream.py`, playback tests |
| Downloads/search | `pikaraoke/lib/download_manager.py` | `pikaraoke/lib/youtube_dl.py`, `pikaraoke/routes/search.py`, download/youtube tests |
| Library scanning | `pikaraoke/lib/library_scanner.py` | `pikaraoke/lib/song_manager.py`, `pikaraoke/lib/karaoke_database.py`, scanner/database tests |
| Preferences | `pikaraoke/lib/preference_manager.py` | `pikaraoke/routes/preferences.py`, preference tests |
| UI template changes | `pikaraoke/templates/` | matching route, `pikaraoke/static/`, route tests |
| Platform/install behavior | `build_scripts/` | `pikaraoke/lib/get_platform.py`, CI smoke tests |
| Metadata/title cleanup | `pikaraoke/lib/metadata_parser.py` | `pikaraoke/routes/metadata_api.py`, `pikaraoke/routes/batch_song_renamer.py`, metadata/batch tests |
| Admin/system control | `pikaraoke/routes/admin.py` | `pikaraoke/lib/current_app.py`, info/admin route tests |
| Translation updates | `build_scripts/update_translations.py` | `pikaraoke/babel.cfg`, `pikaraoke/translations/`, translation tests |

## Important Files

### Root

- `pyproject.toml` defines package metadata, dependencies, console script, and
  coverage omissions.
- `uv.lock` pins resolved dependencies.
- `CLAUDE.md` contains upstream-oriented code style and maintenance guidance.
- `AGENTS.md` contains fork-local agent workflow rules.

### Package

- `pikaraoke/app.py` is imported at process startup and creates global Flask,
  Socket.IO, Babel, and API objects.
- `pikaraoke/karaoke.py` is the runtime coordinator.
- `pikaraoke/constants.py` stores constants such as languages.
- `pikaraoke/version.py` exposes package version state.

### Managers And Helpers

- `args.py`: CLI argument parsing.
- `browser.py`: browser launch/management.
- `current_app.py`: helpers for retrieving app config and broadcasting events.
- `download_manager.py`: serialized download queue and progress.
- `events.py`: lightweight event bus.
- `ffmpeg.py`: FFmpeg discovery and command construction helpers.
- `file_resolver.py`: media path resolution and temp stream naming.
- `karaoke_database.py`: song library database access.
- `library_scanner.py`: filesystem-to-database synchronization.
- `metadata_parser.py`: title cleanup and YouTube ID parsing.
- `playback_controller.py`: now-playing lifecycle.
- `preference_manager.py`: persisted preferences.
- `queue_manager.py`: queue lifecycle.
- `song_manager.py`: song file operations.
- `stream_manager.py`: transcoding and stream readiness.
- `youtube_dl.py`: yt-dlp integration.

## Route Ownership

| Route Module | Main URLs | Calls Into |
|---|---|---|
| `home.py` | `/` | templates and now-playing state |
| `queue.py` | `/queue`, `/enqueue`, `/get_queue`, `/queue/*` | `QueueManager`, `DownloadManager` |
| `search.py` | `/search`, `/autocomplete`, `/preview`, `/download` | `SongManager`, `youtube_dl.py`, `DownloadManager` |
| `files.py` | `/browse`, `/files/delete`, `/files/edit` | `SongManager`, `QueueManager`, admin helpers |
| `controller.py` | `/skip`, `/pause`, `/restart`, `/volume/*`, `/transpose/*` | `PlaybackController`, `Karaoke`, Socket.IO broadcasts |
| `stream.py` | `/stream/*`, `/subtitle/*` | `PlaybackController`, `FileResolver`, temp stream files |
| `splash.py` | `/splash`, `/splash/score_phrases` | `Karaoke`, templates, Raspberry Pi Wi-Fi text |
| `admin.py` | `/auth`, `/logout`, `/sync_library`, `/quit`, `/shutdown`, `/reboot` | admin helpers, `Karaoke`, yt-dlp upgrade |
| `preferences.py` | `/change_preferences`, `/clear_preferences` | `PreferenceManager` |
| `metadata_api.py` | `/metadata/tidy-name`, `/metadata/suggest-names` | `metadata_parser.py` |

## Test Map

| Source Area | High-value Tests |
|---|---|
| CLI args | `tests/unit/test_args.py` |
| Queue manager and routes | `test_queue_manager.py`, `test_queue_routes.py`, `test_queue_reorder.py`, `test_queue_socketio.py`, `test_karaoke_queue.py` |
| Downloads/search | `test_download_manager.py`, `test_youtube_dl.py` |
| Playback/streaming | `test_playback_controller.py`, `test_stream_manager.py`, `test_now_playing_routes.py`, `test_splash_routes.py` |
| Library/database/files | `test_library_scanner.py`, `test_karaoke_database.py`, `test_song_manager.py`, `test_song_list.py`, `test_file_resolver.py` |
| Preferences | `test_preference_manager.py`, `test_preference_routes.py` |
| Metadata/renaming | `test_metadata_parser.py`, `test_batch_renamer.py` |
| Platform/network | `test_get_platform.py`, `test_network.py`, `test_ffmpeg.py` |

### Routes

Routes are grouped by surface: admin, background music, batch renamer,
controller, files, home, images, info, metadata API, now playing, preferences,
queue, search, socket events, splash, and stream.

## Current Testing Shape

The tests are mostly under `tests/unit/` and cover managers, route behavior,
utility seams, and selected app coordination behavior. `tests/conftest.py`
contains lightweight mocks for karaoke, song lists, and playback controller.

## Known Agent Footguns

- `app.py` parses args and initializes global app objects at import time; be
  careful with tests that import it.
- `gevent.monkey.patch_all()` happens early in `app.py`.
- `Karaoke.run()` is a blocking loop; test single methods or mocked loops rather
  than starting the whole runtime unless doing a smoke test.
- `DownloadManager` has a background daemon worker and a shadow pending list;
  tests should control or mock subprocess calls.
- `LibraryScanner` has deletion safety logic. Be careful when changing move or
  missing-file behavior because it protects unmounted song drives.
- Stream IDs include time so repeated plays do not share temp output names.
- Some behavior depends on platform checks, FFmpeg availability, browser launch,
  subprocesses, or filesystem timing. Mock those seams unless a smoke test
  explicitly needs them.
- `rg` failed with "Access is denied" in this Windows environment; PowerShell
  recursive listing was used as fallback.
