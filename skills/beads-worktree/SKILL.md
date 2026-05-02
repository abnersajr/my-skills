---
name: beads-worktree
description: Use when working with Beads in a git worktree setup. Covers BEADS_DIR configuration for bare repos with multiple worktrees, centralized vs per-worktree Beads, and agent coordination across worktrees.
---

# Beads + Worktrees

Project uses bare git clones with worktrees. Beads lives in main worktree. Other worktrees access via `BEADS_DIR`.

## Architecture

```
/main/worktree/          ← Main worktree (has .beads/)
  .beads/                ← Beads database (Dolt)
  
/worktree-a/             ← Agent worktree
  BEADS_DIR → /main/worktree/.beads  (env var)

/worktree-b/             ← Another agent worktree
  BEADS_DIR → /main/worktree/.beads  (env var)
```

All worktrees share ONE Beads database. Issues centralized.

## Setup

### 1. Initialize Beads in Main Worktree

```bash
cd /main/worktree
bd init
```

### 2. Configure BEADS_DIR in Worktrees

Each worktree shell session:

```bash
export BEADS_DIR=/main/worktree/.beads
```

Or add to shell profile:

```bash
# ~/.bashrc or ~/.zshrc
export BEADS_DIR=/path/to/main/worktree/.beads
```

### 3. Verify

```bash
bd ready --json  # Should work from any worktree
```

## Agent Protocol with Worktrees

When dispatching agents to worktrees:

1. Agent reads tasks from centralized Beads: `bd ready --json`
2. Agent claims task: `bd update <id> --claim`
3. Agent creates worktree for code isolation
4. Agent works in worktree, updates progress: `bd update <id> --note "..."`
5. Agent creates PR from worktree to main
6. Agent closes task: `bd close <id> "Done in PR #123"`

## Why BEADS_DIR Over Symlinks

| Approach | Pros | Cons |
|----------|------|------|
| `BEADS_DIR` env var | Clean, standard Beads feature, no filesystem tricks | Must set in each shell session |
| Symlink `ln -s` | Visual, no env var needed | Fragile — breaks on worktree delete/recreate |
| Copy `.beads/` | Fully isolated | Issues fragment across worktrees |

**Use BEADS_DIR.** Beads-native solution.

## Multiple Agents, One Database

Beads uses atomic `--claim` to prevent conflicts:

```bash
# Agent A claims task
bd update bd-a1b2 --claim    # Succeeds

# Agent B tries same task
bd update bd-a1b2 --claim    # Fails — already claimed
```

Always claim before working. Task already claimed? Pick another.

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `bd` not found in worktree | Install beads CLI globally (not per-project) |
| `BEADS_DIR` not working | Verify path exists: `ls $BEADS_DIR` |
| Permission errors | Check file permissions on `.beads/` directory |
| Conflicts between agents | Ensure agents claim tasks before working |