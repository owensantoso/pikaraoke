---
type: architecture-area
id: AREA-LIBRARY
title: Library, Metadata, And Downloads
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

# AREA-LIBRARY - Library, Metadata, And Downloads

## Owns

- The local song catalog and its SQLite persistence.
- Filesystem scanning and database synchronization.
- Song rename/delete/register operations.
- YouTube search, preview, download command construction, and serialized
  download processing.
- Filename metadata parsing, title cleanup, and YouTube ID extraction.

## Does Not Own

- Queue ordering and playback state; see `AREA-PLAYBACK`.
- Browser rendering of search/browse pages; see `AREA-WEB-UI`.
- App process lifecycle; see `AREA-RUNTIME`.

## Stable Invariants

- `KaraokeDatabase` is a pure data layer and should not perform filesystem work.
- `LibraryScanner` owns disk-to-database reconciliation, including move detection
  and the >50% missing-file circuit breaker.
- `SongManager` owns direct file operations and keeps `SongList` plus database
  state synchronized.
- Downloads run through one `DownloadManager` worker thread to reduce rate-limit
  and CPU pressure.
- YouTube filenames support the PiKaraoke `Title---dQw4w9WgXcQ.mp4` pattern and
  yt-dlp `Title [dQw4w9WgXcQ].mp4` pattern.

## Domain Language

| Term | Meaning | Notes |
|---|---|---|
| Song path | Native OS string path to a playable media file. | Stored in SQLite as native strings. |
| Companion file | `.cdg` or `.ass` file sharing the media basename. | Used to detect CDG/ASS formats. |
| Download queue | Internal queue of pending yt-dlp requests. | Separate from the playback queue. |
| Song queue | Playback queue managed by `QueueManager`. | Owned by `AREA-PLAYBACK`. |

## Interfaces / Seams

| Interface | Caller | Owns | Does Not Own |
|---|---|---|---|
| `KaraokeDatabase` | scanner, song manager | SQLite schema, path metadata, integrity checks. | Filesystem scanning. |
| `LibraryScanner.scan()` | `Karaoke.sync_library()` | Diffing disk and DB, moves, inserts, deletes. | UI notifications. |
| `SongManager` | routes, downloads, queue display | Song list, display names, file delete/rename/register. | Playback queue policy. |
| `DownloadManager.queue_download()` | search route | Serialized download requests and progress state. | Search UI rendering. |
| `youtube_dl.py` helpers | search/download manager | yt-dlp commands and parsing. | Persistence or queue policy. |

## Intended Flow

1. On startup, `KaraokeDatabase` opens the library DB in the platform data
   directory.
2. If database paths exist, `SongManager.songs` warms from DB and a background
   sync reconciles disk.
3. If no database paths exist, startup performs a blocking cold scan.
4. Search routes can ask yt-dlp for results or direct preview URLs.
5. Download requests enter `DownloadManager`.
6. Successful downloads emit `song_downloaded`; `SongManager.register_download`
   updates `SongList` and DB.

## Current Mismatch Notes

- `DownloadManager.pending_downloads` is a shadow list around `Queue`; it assumes
  a single worker thread.
- The scanner bypasses the deletion circuit breaker when it detects an intended
  scan-directory move.
- Route-level admin checks protect file edits/deletes, but `SongManager` itself
  assumes callers have already authorized the operation.

## Related Areas

- `AREA-PLAYBACK` - consumes song paths and queue items.
- `AREA-WEB-UI` - exposes browse/search/file-management routes.
