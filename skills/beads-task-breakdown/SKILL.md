---
name: beads-task-breakdown
description: Break features/specs into Beads issues. Create epics, tasks, subtasks; set priorities; establish dependency chains. Triggers: "create issues for", "break down this feature", "plan this in beads".
---

# Beads Task Breakdown

Break features into Beads issues with hierarchy + dependencies.

## Issue Hierarchy

```
Epic (bd-a3f8)                    ← Feature container
  ├── Task (bd-a3f8.1)            ← Single deliverable
  │   ├── Subtask (bd-a3f8.1.1)   ← Atomic unit of work
  │   └── Subtask (bd-a3f8.1.2)
  └── Task (bd-a3f8.2)
      └── Subtask (bd-a3f8.2.1)
```

## Breaking Down a Feature

Given feature spec, create issues in order:

### 1. Create Epic

```bash
bd create "Auth Feature" -p 0 -t epic
# Returns: bd-a1b2
```

### 2. Create Tasks Under Epic

```bash
bd create "Auth: Supabase client setup" -p 0 -t task --parent bd-a1b2
bd create "Auth: Login page UI" -p 0 -t task --parent bd-a1b2
bd create "Auth: Signup flow" -p 1 -t task --parent bd-a1b2
bd create "Auth: Password reset" -p 2 -t task --parent bd-a1b2
```

### 3. Add Dependencies

```bash
# Login page depends on Supabase client
bd dep add bd-a1b2.2 bd-a1b2.1

# Signup depends on Login page (shared components)
bd dep add bd-a1b2.3 bd-a1b2.2
```

### 4. Break Tasks into Subtasks (Optional)

For complex tasks, add subtasks:

```bash
bd create "Auth: Login - email/password form" -p 0 -t task --parent bd-a1b2.2
bd create "Auth: Login - OAuth providers" -p 1 -t task --parent bd-a1b2.2
bd create "Auth: Login - error handling" -p 1 -t task --parent bd-a1b2.2
```

## Priority Guidelines

| Priority | When to Use | Examples |
|----------|-------------|---------|
| P0 | Foundation — blocks everything else | Supabase setup, Auth, Router |
| P1 | Core features — MVP scope | Texts CRUD, Study mode, TTS |
| P2 | Enhancements — improves UX | Search, Dark mode, Keyboard shortcuts |
| P3 | Nice-to-have — future work | Offline cache, Export, Analytics |

## Task Sizing

**Good task:** Completable in one agent session (15-60 min)

| Too Big | Right Size | Too Small |
|---------|------------|-----------|
| "Implement auth" | "Login page UI" | "Add one input field" |
| "Build TTS system" | "WebSpeech provider" | "Import a library" |
| "Study mode" | "Listen & Repeat view" | "Add a button" |

## Dependency Patterns

### Sequential (blocking)
```
A blocks B → B can't start until A is done
```

### Parallel (no deps)
```
A and B can be worked on simultaneously
```

### Related (not blocking)
```
A relates_to B → context link, no blocking
```

## Example: Memo Project Breakdown

```
Epic: Auth (P0)
  ├── Supabase client setup (P0)
  ├── Login page UI (P0) — blocks: Supabase client
  ├── Signup flow (P0) — blocks: Login page
  └── Password reset (P2) — blocks: Login page

Epic: Texts (P0)
  ├── Text repository + types (P0)
  ├── Text list view (P0) — blocks: Text repository
  ├── Text editor (P0) — blocks: Text repository
  └── Markdown preview (P1) — blocks: Text editor

Epic: TTS (P1)
  ├── TTS provider interface (P1)
  ├── WebSpeech provider (P1) — blocks: TTS interface
  ├── Cloud TTS provider (P2) — blocks: TTS interface
  └── TTS player UI (P1) — blocks: WebSpeech provider
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Epic with no tasks | Break epics into deliverable tasks |
| Tasks too big | Split into subtasks if > 1 hour of work |
| No dependencies | Add `bd dep add` for blocking relationships |
| Wrong priority | P0 = can't start anything else without this |
| Missing acceptance criteria | Add notes: `bd update <id> --note "Done when: ..."` |

## Interactive Planning

When breaking down features, consider using interactive methods to gather requirements:

- **grill-me skill**: If the user asks for planning or says "grill me", load the `grill-me` skill to interview them about the plan, resolve ambiguities, and update documentation (CONTEXT.md, ADRs) inline.
- **Clarifying questions**: If the feature spec is ambiguous, ask the user for specifics (priority, dependencies, scope) before creating issues. Use the `question` tool to present structured options.
- **Interactive forms**: For manual user prompts (not automated agents), use `bd create-form` which provides a terminal UI for selecting priorities, labels, parent, etc. This is more user-friendly than CLI flags.
- **Agent preference**: When invoked by an agent (automated), prefer the easier batch CLI commands (`bd create` with flags) for efficiency. Avoid interactive prompts unless necessary.

### When to Ask Questions

- **Unknown priority**: Ask "Is this a foundation (P0), core feature (P1), enhancement (P2), or nice-to-have (P3)?" Use single-choice (`multiple: false`).
- **Missing dependencies**: Ask "What other tasks must be completed before this one can start?" Use multiple-choice (`multiple: true`) if listing existing tasks.
- **Scope unclear**: Ask "Should this be one task or broken into subtasks?" Use single-choice (yes/no or break into subtasks).
- **Custom labels**: Ask "Any custom labels like 'wip', 'frontend', 'auth'?" Use multiple-choice (`multiple: true`) for selecting multiple labels.
- **Use the `question` tool** with appropriate `multiple` parameter: `false` for single-choice (Priority, Type, Scope), `true` for multiple-choice (Labels, Dependencies).

### Interactive Mode Selection

| Scenario | Recommended Approach |
|----------|----------------------|
| Agent creating issues | `bd create` with flags (quick, scriptable) |
| Human creating issues | `bd create-form` or ask clarifying questions |
| Complex feature with many decisions | Use `grill-me` skill to stress-test plan first |
| Batch creation from spec | `bd create --file spec.md` or `bd create --graph plan.json` |