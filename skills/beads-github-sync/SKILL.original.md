---
name: beads-github-sync
description: Use when setting up or troubleshooting Beads to GitHub Issues sync. Covers beads-synced GitHub Action configuration, manual issue creation on GitHub, and hybrid workflows where both Beads and GitHub Issues coexist.
---

# Beads GitHub Sync

Beads issues live in `.beads/` (Dolt database). The `mrf/beads-synced` GitHub Action mirrors them to GitHub Issues as read-only.

## Architecture

```
Beads (.beads/) → GitHub Action → GitHub Issues (read-only mirror)
     ↑                                      ↑
  Source of truth                      Can also create manually
```

Edits happen in Beads. GitHub Issues are a visibility layer.

## Setup: beads-synced Action

Create `.github/workflows/beads-sync.yml`:

```yaml
name: Beads Sync
on:
  push:
    branches: [main]
    paths: ['.beads/**']
jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: mrf/beads-synced@main
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

This syncs on every push that modifies `.beads/`.

## Hybrid Workflow

You can create issues in BOTH places:

- **Beads** → for agent-managed tasks, dependency tracking, structured workflow
- **GitHub Issues** → for human-created bugs, feature requests, community input

### When to create in Beads
- Tasks for agents to execute
- Tasks that need dependency tracking
- Tasks that are part of an epic/plan

### When to create on GitHub
- Bug reports from testing
- Feature ideas not yet planned
- Issues that need community visibility

## Manual Sync

If you need to force a sync:

```bash
# Push .beads/ changes to trigger sync
git add .beads/
git commit -m "chore: update beads issues"
git push
```

## Viewing Synced Issues

- **GitHub:** `https://github.com/<owner>/<repo>/issues` — see mirrored issues
- **Beads CLI:** `bd list --json` — source of truth
- **Beads Viewer:** `https://github.com/mgalpert/beads-viewer` — web UI for Beads

## Labels and Mapping

Beads issues sync with labels:
- Priority → `P0`, `P1`, `P2`, `P3`
- Type → `task`, `bug`, `epic`, `story`
- Status → `open`, `in_progress`, `done`, `closed`

## Troubleshooting

| Issue | Fix |
|-------|-----|
| Issues not syncing | Check `.github/workflows/beads-sync.yml` exists and is on `main` |
| Sync is delayed | Only triggers on push to `main` with `.beads/` changes |
| Duplicate issues | Don't edit GitHub Issues directly — they're read-only mirrors |
| Missing issues | Check `bd list` in CLI — GitHub only shows what's in Beads |
