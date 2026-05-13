---
name: orienting-to-repo
description: Use when starting substantive work in an unfamiliar repo, before diving into task-specific grep or edits — especially when the task touches business logic, public API, build, or infra, or you don't yet know the repo's purpose, stack, conventions, or layout
---

# Orienting to a Repo

## Overview

Before substantive work in a repo you don't already understand, build a brief mental model. Jumping to task-specific grep is a trap: you find symbol X without learning that symbol X is one of three competing implementations and the project is mid-migration. Five minutes of orientation prevents an hour of wrong-direction work.

**Core principle:** Understand what the repo IS before you change it.

## When to Use

- First substantive change in a repo you haven't worked in this session
- After a `/clear` or `/compact` in a repo
- Task touches business logic, public API, build config, or CI

## When NOT to Use

- One-line typo fix
- Renaming a single local variable
- Already oriented in this session (rely on conversation context)
- User explicitly says "skip orientation"

## The Orientation Pass

Run these in parallel where possible. Time-box to ~5 minutes, not 30.

1. **Purpose** — Read `README.md` (intro + setup section). What does this repo DO?
2. **Stack** — Read `package.json` / `pyproject.toml` / `go.mod` / `Gemfile` / `Cargo.toml`. Language, framework, key deps.
3. **Layout** — `ls` the root and 1-2 levels into the main source dir. Note convention: `src/`, `lib/`, `app/`, `pkg/`, etc.
4. **Conventions** — Look for `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, `.editorconfig`, linter configs. These are the rules.
5. **Tests** — Where do they live? What runner? Read one test file to see the style.
6. **CI / build** — Skim `.github/workflows/` or equivalent. What's enforced before merge?
7. **Entry point** — Find and read the main entry file (top of `main.py`, `index.ts`, `cmd/<name>/main.go`).

## Capture the Model — Briefly

After the pass, write a short scratch summary (3-5 bullets) in your reply to the user. Cover:

- Purpose (one sentence)
- Stack
- Where the relevant code for the current task lives
- Conventions to follow (linter, test framework, doc style)
- Any "watch out for X" you noticed (active migration, deprecated dir, etc.)

**Do NOT create a `REPO_MAP.md` file or any persistent artifact.** The map goes stale instantly and lies to future sessions. Keep the model in conversation context only.

## Quick Reference

| Look for | To learn |
|---|---|
| README.md | Purpose, install, basic usage |
| CONTRIBUTING.md / CLAUDE.md / AGENTS.md | Project-specific rules |
| package metadata file | Language + dependencies |
| .github/workflows or .gitlab-ci.yml | What CI enforces |
| Existing test file | Test framework + style |
| `git log -20 --oneline` | What's active right now |

## Common Mistakes

| Mistake | Fix |
|---|---|
| Jumping straight to `grep "<task-symbol>"` | Run the orientation pass first |
| Reading every file under `src/` | Read the entry point, then trace from there |
| Skipping CI config | Skipped CI = surprised by failed build on PR |
| Producing a REPO_MAP.md artifact | Don't. Keep the model in chat context. |
| Spending 30+ minutes orienting | You're over-investing. Time-box ~5 min. |

## Red Flags — STOP and Orient

- About to grep for a task-specific symbol in a repo you haven't read the README of
- About to edit a file in a stack you haven't confirmed (TS vs JS, Python 2 vs 3, etc.)
- "I'll figure out the conventions as I go" — no, find them now
- "The user wants this fast" — fast and wrong is slower than oriented and right
