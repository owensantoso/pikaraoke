---
type: architecture-index
title: Architecture Areas
domain: architecture
status: active
created_at: "2026-04-29 03:35:25 +09:00"
updated_at: "2026-04-29 03:35:25 +09:00"
repo_state:
  based_on_commit: c406a10
  last_reviewed_commit: c406a10
---

# Architecture Areas

Stable boundaries for PiKaraoke agent work. Use these IDs in plans, session
logs, commit messages, and subagent handoffs when a task touches one of these
surfaces.

| Area | Owns |
|---|---|
| `AREA-RUNTIME` | App startup, CLI configuration, Flask/Socket.IO composition, process lifecycle. |
| `AREA-LIBRARY` | Song discovery, metadata parsing, SQLite library state, file rename/delete, YouTube downloads. |
| `AREA-PLAYBACK` | Queue-to-playback flow, FFmpeg preparation, stream serving, now-playing state. |
| `AREA-WEB-UI` | Server-rendered pages, browser assets, admin/control/search/browse routes, Socket.IO events. |

Read [orientation/ARCHITECTURE.md](../orientation/ARCHITECTURE.md) first for the
system overview, then the area doc for the boundary you are touching.
