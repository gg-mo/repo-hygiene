---
name: keeping-docs-fresh
description: Use when changing a public API surface — renamed export, changed signature/return type, new or removed flag/route/env var, schema migration, deprecation — before committing, so README, docs/, CHANGELOG, and inline references stay coherent with the code
---

# Keeping Docs Fresh

## Overview

Any change to the public API surface should propagate to the docs that describe it — in the same commit. Otherwise the README quietly starts lying to readers. The honest baseline from agents is "ship it = commit whatever's staged," which silently leaves stale docs in the repo.

**Core principle:** Code change without doc change = doc lies. Scan before you commit.

## What Counts as a Public API Change

Trigger this skill when any of these happen:

- Renamed an exported function, class, type, or constant
- Changed the signature, return type, or behavior contract of an exported item
- Added or removed a CLI flag, env var, or config key
- Added or removed an HTTP route, GraphQL field, or RPC method
- Changed a database schema (added/removed/renamed column)
- Marked a public symbol deprecated or removed it
- Changed default behavior in a way users would notice

**NOT triggered by:** Internal refactor with no exported surface change, comment tweaks, formatting, test-only changes.

## The Scan

Run BEFORE committing. One ripgrep catches most references:

```bash
# Old symbol name across docs + non-source files
rg -n 'getUserById' README.md docs/ CHANGELOG.md 2>/dev/null
# Broader sweep including inline comments in code
rg -n 'getUserById' .
```

For each hit, decide:
- **Update** — the doc should reflect the new name / signature / behavior
- **Remove** — the doc described a thing that no longer exists
- **Keep** — historical context (release notes, ADRs); leave alone

If the scan returns zero hits, say so explicitly ("No doc references to `<old>` found, safe to commit") rather than skipping the step silently.

## What to Update

| File | When |
|---|---|
| `README.md` | Usage examples or feature list mention the change |
| `docs/**/*.md` | Always scan; update matching references |
| `CHANGELOG.md` (if present) | Add an entry; mark breaking changes as such |
| `openapi.yaml` / GraphQL schema | HTTP / GraphQL surface changed |
| `--help` text / man pages | CLI flags changed |
| `.env.example` | Env vars added or removed |
| Inline `//` or `#` comments | Comments name the old symbol |

## Quick Example

You renamed `getUserById(id: string): User` to `fetchUser(id: string): Promise<User>`.

```text
$ rg -n 'getUserById' .
README.md:42: `getUserById("abc")` example
docs/api.md:18: function reference
src/legacy/migrate.ts:99: comment
```

Update each: rewrite README example to `await fetchUser("abc")`, update `docs/api.md` to the new name and async, fix the comment in `migrate.ts`, add a CHANGELOG entry like **BREAKING:** Renamed `getUserById` → `fetchUser`; now returns `Promise<User>`. Then commit all of it together.

## Common Mistakes

| Mistake | Fix |
|---|---|
| "Ship it" → commit immediately | Scan first. Doc updates ride in the same commit. |
| Updating README only, missing `docs/` | Use `rg` from repo root, not selective reads |
| Forgetting CHANGELOG when one exists | Every public-surface change deserves an entry |
| Soft-pedaling a breaking change | If it breaks callers, mark it **BREAKING** |
| "I'll update docs later" | Later = never. Same commit. |
| Spotting a stale reference and thinking "out of scope" | If it's stale because of YOUR change, it's in scope |

## Red Flags — STOP

- About to commit a renamed export without running the scan
- "The user said ship it" — ship still means doc-coherent ship
- "It's just a typo" — not a typo if the function's contract changed
- "I'll update docs later" — almost always becomes never
- Saw a stale reference in `docs/` and rationalized it as out of scope
