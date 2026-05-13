---
name: using-repo-hygiene
description: Use when starting any conversation in a repo — establishes the repo-hygiene skill set and the moments each sub-skill applies
---

<SUBAGENT-STOP>
If you were dispatched as a subagent to execute a specific task, skip this skill.
</SUBAGENT-STOP>

# Using Repo Hygiene

This plugin keeps a codebase healthy as you work in it. Each sub-skill triggers on a specific moment — invoke it then via the `Skill` tool.

**Important override:** This skill set INVERTS the usual "no comments unless WHY is non-obvious" default. In a repo using this plugin, docstrings on functions and file headers ARE expected. Quality matters (see `repo-hygiene:writing-docstrings`); silence does not.

## Trigger Table

| When this happens | Invoke |
|---|---|
| Starting substantive work in a repo you don't already understand | `repo-hygiene:orienting-to-repo` |
| Writing or modifying a function, class, or source file | `repo-hygiene:writing-docstrings` |
| About to add lines to a file >400 lines, or your edit pushes one past that | `repo-hygiene:keeping-files-small` |
| Changing a public API (renamed export, new flag, new route, changed signature) | `repo-hygiene:keeping-docs-fresh` |
| Adding business logic, branching code, or fixing a bug | `repo-hygiene:testing-new-code` |
| Drafting any commit message | `repo-hygiene:writing-commit-messages` |
| User asks to commit, merge, or push | `repo-hygiene:gating-commits` |
| User asks to "clean up" or apply repo-hygiene to an existing codebase | `repo-hygiene:cleaning-existing-codebase` |

## Priority

1. **User instructions win.** If the user says "skip the orientation," skip it.
2. **Coordinate with peer skill sets.** If `superpowers:test-driven-development` is already in play, `testing-new-code` is satisfied by that workflow.
3. **Otherwise apply** at the triggers above.

## Bypass

A user can bypass a hygiene skill for one turn with `#hygiene-skip <reason>` in their message. Acknowledge the bypass and proceed.
