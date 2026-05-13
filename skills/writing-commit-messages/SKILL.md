---
name: writing-commit-messages
description: Use when drafting any commit message — sets format (imperative subject under 72 chars, explanatory body when the why isn't obvious from the diff) and prevents the "Fix X" one-liner reflex for non-trivial fixes
---

# Writing Commit Messages

## Overview

A commit message is the only place future-you (or future-them) finds out WHY a change happened. The diff shows what; the message has to explain why. The honest agent default is a one-liner subject and no body — fine for trivial changes, lossy for everything else.

**Core principle:** The message answers "why," not "what." The diff already shows what.

## The Format

```
<subject — imperative, under 72 chars, capitalize first letter, no period>
<blank line>
<body — wrap at 72 cols, explain WHY this change exists>

<blank line, optional>
<trailer — e.g. Fixes #123, Co-Authored-By: ...>
```

## Subject Line

- **Imperative mood:** `Fix login failure for apostrophe usernames` — not `Fixed`, not `Fixes`, not `Fixing`.
- **Under 72 chars** (50 ideal). If you need more, the body should be doing that work.
- **No trailing period.**
- **Capitalize the first letter.**

## When To Write a Body

Write a body when ANY of these are true:

- The change fixes a bug — the body explains the root cause and the failure mode
- The change is non-obvious from the diff alone
- The change has a non-obvious motivation (compliance, perf regression, deprecation)
- The change is part of a larger initiative — link to the spec or parent ticket
- The change has subtle behavior (silent failure, edge case, race-condition fix)

Skip the body for: pure renames, formatting / lint fixes, dep version bumps with no behavior change, typo fixes in non-code files.

## Body Content

Include:

- **Symptom / motivation** — what was wrong, or why this exists
- **Root cause** — why the bug existed (for fixes)
- **Approach** — only if non-obvious (`We chose X over Y because Z`)
- **Caveats / follow-ups** — known limitations, deferred work

Leave out: play-by-play of the diff, redundant restatement of the subject, marketing speak.

## Before / After

❌ **One-liner reflex (loses the why):**
```
Fix login bug
```

❌ **What-not-why (diff already shows this):**
```
Update src/auth/validator.ts

Removed the .replace() call in validator.ts.
```

✅ **Subject + body that explains the why:**
```
Fix silent login failure for usernames with apostrophes

The validator was stripping apostrophes via .replace(/'/g, '') before
the database lookup, so users like "O'Brien" were queried as "OBrien"
and silently never matched. Removing the replacement passes the
username through unchanged.

Fixes #482
```

## Common Mistakes

| Mistake | Fix |
|---|---|
| Past-tense subject (`Fixed apostrophe bug`) | Imperative: `Fix apostrophe bug` |
| Subject longer than 72 chars | Move detail to the body |
| Body that restates the subject | Cut it; explain why instead |
| Body that walks through the diff | Cut it; reader can read the diff |
| No body on a non-trivial bug fix | Add one — what was broken, why, how this fixes it |
| `Misc improvements` / `Various fixes` | One logical change per commit. Split first. |
| Mentioning a ticket number without context | Either explain in the body or just leave the trailer |

## Red Flags — STOP

- Subject starts with a past-tense verb
- Subject is your only content for a non-trivial fix
- Body is a list of what you changed (= what the diff shows)
- Multiple unrelated changes in one commit — split first
- About to commit a bug fix with no body explaining the root cause
