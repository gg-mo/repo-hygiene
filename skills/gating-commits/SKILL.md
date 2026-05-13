---
name: gating-commits
description: Use when about to commit, merge, or push — runs verification (tests, typecheck, lint), confirms focus, and checks sibling hygiene (docs, tests, docstrings) BEFORE the commit lands, so "user said commit" doesn't bypass quality
---

# Gating Commits

## Overview

"User said commit" is not authorization to skip verification. The honest baseline from agents is: do git triage, draft a message, stage, commit — no test/typecheck/lint in between. This skill changes the default to **verify first, commit second**.

**Core principle:** A commit is a promise that what's in it works. Don't ship that promise on faith.

## When To Use

- User says "commit" / "ship" / "land" / "merge" / "push"
- Just finished a multi-file change
- About to call `git commit` for any reason

## The Gate

Run these BEFORE staging or committing. Stop at the first failure.

1. **Identify the verification commands.** Read `package.json`, `pyproject.toml`, `Makefile`, `.github/workflows/`, or `CLAUDE.md`. Typical targets:
   - Tests: `npm test`, `pytest`, `go test ./...`, `cargo test`
   - Types: `tsc --noEmit`, `mypy`, `pyright`
   - Lint: `npm run lint`, `ruff check`, `golangci-lint run`
2. **Run them.** Read the full output. Check exit codes.
3. **If anything fails:** Fix or revert. Do NOT commit failing code with intent to fix in the next commit.
4. **Sibling-skill checks:**
   - Changed a public API surface? Did `repo-hygiene:keeping-docs-fresh` run?
   - Added business logic? Did `repo-hygiene:testing-new-code` run?
   - Added new functions/classes/files? Did `repo-hygiene:writing-docstrings` apply?
5. **Focus check** — `git diff --stat HEAD`. Are all changes related to the stated commit purpose? If unrelated changes snuck in, split them.
6. **Then** stage specific files (no blanket `git add .`) and commit with a clear message.

## What Counts as "Verified"

| Check | Sufficient evidence |
|---|---|
| Tests pass | Test command output: 0 failures, exit 0 |
| Types clean | Type-checker output: 0 errors |
| Lint clean | Linter output: 0 errors (warnings: use judgment) |
| Build succeeds | Build command exits 0 |

"Looks fine," "should pass," and "I'm confident" do not count.

## What if the Repo Has No Verification Setup?

Tell the user explicitly:

> "I don't see test/lint/typecheck commands in this repo. I can commit as-is, or set up a minimal check first. Which do you want?"

Defer to the user. Silent skipping is the failure mode.

## Commit Message Hygiene

- **Subject:** imperative mood, under 72 chars (`Fix foo when bar`, not `Fixed foo` or `Fixing foo`)
- **Body:** explain WHY, not what (the diff shows what)
- **One logical change per commit.** Bundling unrelated fixes makes review and bisect harder.

## Common Mistakes

| Mistake | Fix |
|---|---|
| "User said commit" → commit | Authorization ≠ verification |
| Tests only, not types | Linter ≠ compiler ≠ test runner; check each |
| `git add .` blanket stage | Stage specific files |
| "Tests failed but they're unrelated" | Either fix or split into a separate commit that flags the breakage |
| Giant diff, multiple unrelated changes | Split. One logical change per commit. |
| Skipping doc/test sibling checks | Re-read the trigger list above |
| `--no-verify` to bypass pre-commit hooks | Investigate why the hook is firing; don't bypass |

## Red Flags — STOP

- About to type `git commit` without fresh test/lint/typecheck output in context
- "User said commit so it's their call" — verification is YOUR job before the commit
- Diff shows unrelated changes — split first
- About to use `--no-verify`
- "I'll fix the failing test in the next commit" — no; fix or revert now
