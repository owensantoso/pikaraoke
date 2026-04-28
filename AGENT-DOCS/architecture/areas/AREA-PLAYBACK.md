---
type: architecture-area
id: AREA-PLAYBACK
title: Queue, Playback, And Streaming
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

# AREA-PLAYBACK - Queue, Playback, And Streaming

## Owns

- Song queue policy and queue mutation.
- Current playback state and now-playing payloads.
- Main queue-to-playback loop.
- File resolution for playback, including CDG zip/MP3 handling and ASS subtitle
  discovery.
- FFmpeg stream preparation and stream lifecycle cleanup.
- HTTP stream serving for HLS and MP4 playback.

## Does Not Own

- YouTube downloading and library scanning; see `AREA-LIBRARY`.
- Route permissions and page rendering; see `AREA-WEB-UI`.
- App startup and browser launch; see `AREA-RUNTIME`.

## Stable Invariants

- `QueueManager` owns the queue list and emits queue/now-playing events after
  mutations.
- `PlaybackController` owns now-playing state and delegates stream preparation
  to `StreamManager`.
- `StreamManager` owns FFmpeg process state and temporary output preparation.
- `FileResolver` creates process-scoped temporary stream names and handles CDG,
  ZIP, MP3, MP4/WebM, and ASS companion discovery.
- Stream routes only mark playback started when the requested stream ID matches
  the current song URL, avoiding stale requests.
- `delete_tmp_dir()` is part of playback cleanup and handles Windows file-lock
  tolerance.

## Domain Language

| Term | Meaning | Notes |
|---|---|---|
| Queue item | Dict with `user`, `file`, `title`, and `semitones`. | Produced by `QueueManager.enqueue`. |
| Now playing | Current song state exposed to splash/browser clients. | Owned by `PlaybackController`, enriched by `Karaoke.get_now_playing()`. |
| Stream UID | Per-play unique ID derived from file path and time. | Prevents repeat plays from colliding in temp files. |
| HLS stream | Default streaming mode with `.m3u8`, fMP4 init, and `.m4s` segments. | Served by `routes/stream.py`. |
| Progressive MP4 | Alternate streaming mode or full-file range route. | Used for compatibility cases. |

## Interfaces / Seams

| Interface | Caller | Owns | Does Not Own |
|---|---|---|---|
| `QueueManager.enqueue()` | routes, transpose, randomizer | Duplicate checks, user limits, fair queue insertion. | Downloading missing files. |
| `QueueManager.pop_next()` | `Karaoke.run()` | Removing the next queued item without flicker events. | Starting playback. |
| `PlaybackController.play_file()` | `Karaoke.run()` | Now-playing state and stream readiness wait. | Queue selection. |
| `StreamManager.play_file()` | playback controller | Transcode/copy decisions, FFmpeg process. | HTTP response serving. |
| `routes/stream.py` | browser player | Serving playlist, segments, MP4, background video, subtitles. | FFmpeg command construction. |

## Intended Flow

1. User or admin enqueues a song.
2. `Karaoke.run()` waits for no active playback and a non-empty queue.
3. The run loop resets now-playing state, waits splash delay, and pops the next
   queue item.
4. `PlaybackController.play_file()` validates the file and asks `StreamManager`
   to prepare a stream.
5. Browser client requests the stream URL; stream route marks the song started.
6. Playback ends, skip/restart/transpose occurs, or timeout/error resets state
   and cleans temp files.

## Current Mismatch Notes

- `PlaybackController.play_file()` waits up to about 10 seconds for a client to
  connect before treating the stream as unplayable.
- Some stream routes block briefly waiting for generated files to appear.
- `restart()` toggles state and broadcasts restart to clients; it does not
  rebuild the stream.
- Transpose re-enqueues the current file at the front with semitones and skips
  the current playback.

## Related Areas

- `AREA-LIBRARY` - provides song paths and downloaded media.
- `AREA-WEB-UI` - sends control actions and receives Socket.IO updates.
