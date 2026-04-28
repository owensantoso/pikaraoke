# Session Logs

Session logs are compact receipts for meaningful agent work. They help future
agents recover what happened without reading chat history.

## When To Add One

Add a session log after:

- docs initialization or restructuring
- repo orientation that future work depends on
- non-trivial implementation
- debugging with important findings
- verification runs that establish a baseline

## Naming

Use:

```text
YYYY-MM-DD-short-session-title.md
```

## Template

```markdown
---
type: session-log
title:
created_at:
updated_at:
branch:
repo_state:
  based_on_commit:
  ending_commit:
---

# <Session Title>

## Goal

## Actions

## Decisions

## Validation

## Next
```

## Logs

- [2026-04-29 - Agent Docs Initialization](2026-04-29-agent-docs-initialization.md)
