---
name: cleaning-existing-codebase
description: Use when the user asks to "clean up", "apply repo-hygiene to", or "fix the hygiene of" an existing codebase — orchestrates the other hygiene skills in a phased, reviewable order rather than firing a 200-file mega-diff
---

# Cleaning an Existing Codebase

## Overview

When a user asks to apply hygiene to an existing project, the seductive path is to start with the visible, mechanical work — docstrings on every file, README rewrite, dep pruning — and ship a 200-file diff. That optimizes for *appearing* thorough; it does NOT optimize for *being* useful. The god-class is usually the real liability; everything else is decoration.

**Core principle:** Survey, propose, get buy-in, work in small reviewable phases. Don't type any cleanup code until the user has approved a plan.

## When To Use

- User asks to "clean up" / "apply repo-hygiene" / "fix the hygiene of" an existing project
- Onboarding into a long-running codebase with obvious hygiene debt
- Tech-debt sprint where hygiene is the focus

## The Process

### 1. Survey

Invoke `repo-hygiene:orienting-to-repo` first. Then quantify the hygiene gaps with concrete numbers:

- **File-size offenders** — files >500 lines, ranked. `find . -type f \( -name '*.py' -o -name '*.ts' -o -name '*.go' \) -exec wc -l {} + | sort -rn | head -20`
- **Docstring coverage** — sample 10 random functions across the codebase; how many have useful docstrings?
- **Test coverage** — count test files vs source files. Run the suite if possible; capture pass rate and gaps.
- **Stale docs** — `git log -1 --format=%cs README.md CHANGELOG.md`; spot-check the README against current code
- **Dead-code candidates** — unused imports, unreferenced exports. Don't act yet; just note.
- **Dep hygiene** — unused deps in `requirements.txt` / `package.json`

Output a concise inventory (~10 bullets with numbers). NOT a 5-page audit.

### 2. Propose a Phased Plan

Show the plan to the user BEFORE touching code. Default ordering, by leverage and dependency:

| Phase | Why this position |
|---|---|
| **0 — Safety net** | Add a smoke test or characterization test for the biggest module BEFORE any refactor. Refactoring without tests is reckless. |
| **1 — Cheap visibility wins** | Fix README, prune unused deps, add CHANGELOG entry. Reviewable in minutes; signals progress. |
| **2 — Architecture** | Decompose god-files / god-classes. Highest risk; needs Phase 0 done first. |
| **3 — Documentation sweep** | Docstrings across files. Last, so docs describe the SHAPE AFTER restructuring, not the moving target. |
| **4 — Test backfill** | Coverage floor on changed code. Use `repo-hygiene:testing-new-code` per file. |

Adjust per the survey. Some projects skip Phase 2 entirely; others ARE Phase 2.

**Wait for explicit user approval before starting Phase 0.**

### 3. Execute, One Phase Per PR

- One phase = one PR. Not one mega-PR.
- Inside a phase, many small commits (one logical change each). Use `repo-hygiene:writing-commit-messages`.
- After each phase, ask the user to review and merge before starting the next.
- If a phase surfaces more issues, NOTE them for a future phase. Don't expand the current scope.

### 4. Use the Other Hygiene Skills as Tools

This skill orchestrates; the others do the work:

- File-size offenders → `repo-hygiene:keeping-files-small` for split proposals
- New code added during refactor → `repo-hygiene:writing-docstrings`
- Public-API changes during refactor → `repo-hygiene:keeping-docs-fresh`
- Each commit → `repo-hygiene:gating-commits` and `repo-hygiene:writing-commit-messages`

## What NOT To Do

| Anti-pattern | Why bad |
|---|---|
| Start with docstring sweep | Highest visibility, lowest leverage. Decorates without fixing. |
| Refactor a god-class before adding tests | Reckless. You'll break things you can't detect. |
| One mega-PR with everything | Unreviewable. Reverting one issue reverts everything. |
| Skip the survey, just start fixing | You'll fix visible issues and miss load-bearing ones. |
| Expand scope mid-phase ("while I'm here…") | Phases stop being focused. Note and move on. |

## Red Flags — STOP

- About to write docstrings on file #1 of 47 with no plan presented
- Refactoring a large file with no test coverage in place first
- Bundling architecture + docs + deps in one commit
- "Appearing thorough" — if your gut says you're optimizing for looking busy over being useful, you are. Pause and check the plan.
- More than ~10 files changed in a single commit during cleanup

## The Bottom Line

A good cleanup leaves the codebase reviewable. A bad cleanup ships a diff so big nobody reads it and no one can revert any single piece of it.
