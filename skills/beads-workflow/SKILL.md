---
name: beads-workflow
description: 'Use when working with Beads issue tracker (bd CLI). Covers task lifecycle: ready -> claim -> work -> update -> close. Use when user says "check beads", "what''s next", "claim a task", "update issue", or when agent needs to find work to do.'
---

# Beads Workflow

Beads (`bd`) = distributed graph issue tracker for AI agents. Core task lifecycle below.

## Setup Check

Verify Beads initialized before using `bd`:

```bash
bd ready --json 2>/dev/null || echo "Beads not initialized. Run: bd init"
```

`BEADS_DIR` set -> all commands use that path. Otherwise `bd` discovers `.beads/` from git repo root.

## Task Lifecycle

```
bd ready -> bd update <id> --claim -> [work] -> bd update <id> --status done -> bd close <id> "reason"
```

### 1. Find Work

```bash
bd ready --json          # Tasks with no open blockers, JSON output
bd ready                 # Human-readable output
bd ready -p 0            # Only P0 tasks
```

### 2. Claim a Task

```bash
bd update <id> --claim   # Atomic: sets assignee + in_progress status
```

**Always claim before starting work.** Prevents duplicate effort in multi-agent setups.

### 3. Update Progress

```bash
bd update <id> --status in_progress   # Already done by --claim
bd update <id> --note "Implemented auth service, writing tests now"
bd update <id> --add-tag wip
```

### 4. Close a Task

```bash
bd close <id> "Brief summary of what was done"
bd close <id> "Fixed in commit abc123" --link abc123
```

### 5. Block/Unblock

```bash
bd dep add <child> <parent>           # child blocks parent
bd dep add <child> <parent> --type blocks
bd dep remove <child> <parent>
```

## Creating Issues

```bash
bd create "Title" -p 0 -t task        # P0 task
bd create "Title" -p 1 -t bug         # P1 bug
bd create "Title" -p 2 -t epic        # Epic (container)
bd create "Title" --parent <epic-id>  # Child of epic
```

Priority levels: `0` (critical), `1` (high), `2` (medium), `3` (low)

Types: `task`, `bug`, `epic`, `story`, `message`

## Viewing Issues

```bash
bd show <id>              # Full details + audit trail
bd list                   # All issues
bd list --status open     # Open only
bd list --assignee me     # My tasks
bd list --json            # JSON output for parsing
```

## Agent Protocol

When dispatched as agent:

1. Run `bd ready --json` -> find available tasks
2. Pick highest priority (P0 > P1 > P2 > P3)
3. `bd update <id> --claim` -> atomic claim
4. Do work
5. `bd update <id> --note "progress update"` periodically
6. `bd close <id> "summary"` when done
7. Go to step 1

**Never work on unclaimed task.** Claim first.

## JSON Output

Use `--json` for agent consumption:

```bash
bd ready --json | jq '.[] | {id, title, priority}'
bd show <id> --json | jq '.dependencies'
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Working without claiming | Always `bd update <id> --claim` first |
| Closing without summary | Always provide brief reason |
| Not checking blockers | Run `bd ready` — don't assume tasks unblocked |
| Ignoring dependencies | Check `bd show <id>` for blockers before claiming |