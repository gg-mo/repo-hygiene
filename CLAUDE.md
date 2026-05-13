# Repo Hygiene — Contributor Guidelines

## What this plugin is

`repo-hygiene` is a small, opinionated skill set that keeps codebases healthy as coding agents work in them. It complements (does not replace) `superpowers`. Where `superpowers` provides workflow disciplines (brainstorming, TDD, debugging, planning), this plugin provides the lower-level hygiene that should hold across every change:

- Orient before you edit
- Docstring what you write
- Update docs when you change public surface
- Test what you ship
- Verify before you commit

Each skill is a separate file under `skills/<name>/SKILL.md`. The bootstrap (`using-repo-hygiene`) is injected at session start via the `hooks/session-start` hook on every supported harness.

## If you're editing a skill

Skills are documentation that shapes agent behavior. Edits should be tested, not just written:

1. **RED — baseline test.** Dispatch a subagent with a representative scenario WITHOUT the skill loaded. Document what it does naturally.
2. **GREEN — write the skill.** Address the specific failure modes you saw.
3. **REFACTOR — close loopholes.** Dispatch again WITH the skill, look for rationalizations, plug them.

Don't commit edits to skills that weren't tested. See `superpowers:writing-skills` for the full methodology.

## What does NOT belong in this plugin

- Skills tied to a specific language, framework, or product. Those go in their own plugin.
- Anything `superpowers` already does well. Don't reinvent TDD, brainstorming, or planning.
- Hard rules that should be linter / CI checks. If a regex can enforce it, it doesn't need a skill.
- Personal preferences. Skills should generalize across users.

## Plugin / platform plumbing

| Platform | Manifest |
|---|---|
| Claude Code | `.claude-plugin/plugin.json` + `.claude-plugin/marketplace.json` |
| Codex (CLI + App) | `.codex-plugin/plugin.json` |
| Cursor | `.cursor-plugin/plugin.json` + `hooks/hooks-cursor.json` |
| OpenCode | `.opencode/plugins/repo-hygiene.js` + `package.json` `main` field |
| Gemini CLI | `gemini-extension.json` + `GEMINI.md` |
| Copilot CLI | Reuses `hooks/` (detects `COPILOT_CLI=1` env var) |

The SessionStart hook (`hooks/session-start`) reads `skills/using-repo-hygiene/SKILL.md` and emits it as `additionalContext` to whichever harness invoked it. The shape of the JSON output is platform-dependent and detected at runtime.

## Bumping the version

Update `version` in all of:

- `.claude-plugin/plugin.json`
- `.claude-plugin/marketplace.json` (plugins[0].version)
- `.codex-plugin/plugin.json`
- `.cursor-plugin/plugin.json`
- `gemini-extension.json`
- `package.json`

Use semver.

## Filing issues / PRs

- One problem per PR.
- For skill changes: include before/after subagent test transcripts. Untested skill edits will be reverted.
- For new skills: justify that the gap isn't covered by an existing skill (in this plugin or `superpowers`).
- For platform plumbing: include a session transcript from the affected harness showing the bootstrap loaded.
