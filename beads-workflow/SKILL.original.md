---
name: beads-workflow
description: Use when working with Beads issue tracker (bd CLI). Covers task lifecycle: ready → claim → work → update → close. Use when user says "check beads", "what's next", "claim a task", "update issue", or when agent needs to find work to do.
---

# Beads Workflow

Beads (`bd`) is a distributed graph issue tracker for AI agents. This skill covers the core task lifecycle.

## Setup Check

Before using `bd`, verify Beads is initialized:

```bash
bd ready --json 2>/dev/null || echo "Beads not initialized. Run: bd init"
```

If `BEADS_DIR` is set, all commands use that path. Otherwise, `bd` discovers `.beads/` from git repo root.

## Task Lifecycle

```
bd ready → bd update <id> --claim → [work] → bd update <id> --status done → bd close <id> "reason"
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

**Always claim before starting work.** This prevents duplicate effort in multi-agent setups.

### 3. Update Progress

```bash
bd update <id> --status in_progress   # Already done by --claim
bd update <id> --note "Implemented auth service, writing tests now"
bd update <id> --add-tag wip
```

### 4. Close a Task

```bash
./scripts/bd-finish.sh <id> "Brief summary"    # Full automation (recommended)
# OR
bd close <id> "Brief summary"                  # Manual close only
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

**Always sync after creating issues** so other agents can see them:

```bash
bd github sync                         # Sync issues with GitHub
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

When dispatched as an agent:

1. Run `bd ready --json` to find available tasks
2. Pick highest priority task (P0 > P1 > P2 > P3)
3. `bd update <id> --claim` to atomically claim it
4. Do the work:
   - Commit after each logical unit: `git add -A && git commit -m "feat(scope): description"`
   - Post-commit hook auto-pushes every 3 commits
5. `bd update <id> --note "progress update"` after each commit
6. `./scripts/bd-finish.sh <id> "summary"` when done:
   - Final git push
   - `bd close` (closes the bead)
   - `bd dolt push` (syncs beads)
   - Auto-PR if task has `needs-pr-review` label
7. Go to step 1

**Never work on an unclaimed task.** Always claim first.

### Commit Rules

- Commit after each logical unit (component, hook, test, etc.)
- Use conventional commits: `type(scope): description`
- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`
- Scopes: `auth`, `folders`, `texts`, `tts`, `recall`, `study`, `search`, `sessions`, `shared`
- Let hooks handle pushing - don't run `git push` manually

### PR vs Direct Push

| Scenario | Behavior |
|----------|----------|
| Task has `needs-pr-review` label | `bd-finish.sh` creates PR automatically |
| Task has no special label | Direct push to main |

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
| Closing without summary | Always provide a brief reason |
| Not checking blockers | Run `bd ready` — don't assume tasks are unblocked |
| Ignoring dependencies | Check `bd show <id>` for blockers before claiming |
