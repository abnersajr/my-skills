---
name: pubweb-write-commits
description: Write commit messages for busbud/public-website (pubweb). Enforces Conventional Commits with project-specific scopes and rules. Use when writing, reviewing, or suggesting commit messages in the pubweb repo.
---

# pubweb-write-commits

## Format

```
type(scope): short description
```

- `scope` optional. No period. Lowercase description. Subject ≤72 chars. Imperative mood.

## Types

| Type | When |
|------|------|
| `feat` | new capability, new component, new flag |
| `fix` | bug fix |
| `chore` | tooling, config, deps, formatting, cleanup |
| `refactor` | restructure, no behavior change |
| `perf` | performance improvement |
| `test` | add/fix tests only |
| `docs` | doc-only changes |
| `ci` | CI/workflow changes |
| `revert` | revert prior commit |

## Common Scopes

Use module/feature area. Examples from codebase:

`checkout` `async-cart` `results` `header` `buson` `buson-home` `whitelabel` `xp` `adyen` `eslint` `url-builder` `aasa`

No scope = cross-cutting or repo-wide change.

## Rules

- No issue refs in commits — those go in PR body only
- No AI attribution ("co-authored by", "generated with", etc.)
- No vague verbs: ~~adjust, improve, handle, start~~ → use: add, fix, remove, extract, gate, scope, disable, hide, populate, wire, prefer
- No "this commit" / "this PR" prefix
- Body optional — use only when WHY is non-obvious

## Real Examples

```
feat(async-cart): disable pix submit button while purchase request is in flight
fix(async-cart): populate selected seats in Redux after cart creation
refactor(checkout): extract tryRedirectIfCartAlreadyBought helper
perf: lazy-load Adyen checkout CSS on async cart mount
fix(buson): default locale to pt-BR on www2.buson.com.br
chore(adyen): upgrade web drop-in to 6.32.0
feat(results): gate ANTT banner on Buson e-commerce behind SHOW_ANTT_BANNER XP
fix(url-builder): preserve whitelabel query param in production SSR
```

## Anti-patterns

```
# BAD
feat: adds new feature for the checkout flow (#12345)  ← issue ref + vague
chore: improve stuff                                    ← vague verb
fix: fix the bug                                        ← redundant
feat: Add User Authentication                           ← not lowercase
```
