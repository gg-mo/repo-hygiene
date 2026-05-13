# Repo Hygiene

A small, opinionated skill set that keeps codebases healthy as coding agents work in them. Modeled on [Superpowers](https://github.com/obra/superpowers): one bootstrap skill is injected at session start, and five sub-skills auto-trigger at the right moments.

It complements Superpowers — Superpowers gives you the workflow disciplines (brainstorming, TDD, debugging, planning); `repo-hygiene` gives you the lower-level hygiene that should hold across every change:

- Orient before you edit
- Docstring what you write
- Keep files from sprawling
- Update docs when you change public surface
- Test what you ship
- Write commit messages that explain WHY
- Verify before you commit
- Clean up existing codebases in reviewable phases

## How it works

A SessionStart hook injects [`skills/using-repo-hygiene/SKILL.md`](skills/using-repo-hygiene/SKILL.md) into the system context of every conversation. That bootstrap tells the agent which sub-skill applies at which moment. The agent then invokes the relevant sub-skill via the harness's `Skill` (or equivalent) tool when its trigger fires.

The skills don't trigger on every keystroke — they trigger at meaningful moments: starting work in a new repo, writing a new function, changing a public API, committing.

## Skill catalog

| Skill | Triggers when |
|---|---|
| [`using-repo-hygiene`](skills/using-repo-hygiene/SKILL.md) | Bootstrap — loaded into every session |
| [`orienting-to-repo`](skills/orienting-to-repo/SKILL.md) | Starting substantive work in an unfamiliar repo |
| [`writing-docstrings`](skills/writing-docstrings/SKILL.md) | Writing or modifying any function, method, class, or source file |
| [`keeping-files-small`](skills/keeping-files-small/SKILL.md) | About to add lines to a file already >400 lines, or your edit pushes one past that |
| [`keeping-docs-fresh`](skills/keeping-docs-fresh/SKILL.md) | Changing a public API surface (renames, signature changes, new flags/routes/env vars, schema migrations) |
| [`testing-new-code`](skills/testing-new-code/SKILL.md) | Adding business logic, branching code, public API, or fixing a bug |
| [`writing-commit-messages`](skills/writing-commit-messages/SKILL.md) | Drafting any commit message |
| [`gating-commits`](skills/gating-commits/SKILL.md) | About to commit, merge, or push |
| [`cleaning-existing-codebase`](skills/cleaning-existing-codebase/SKILL.md) | User asks to "clean up" or apply repo-hygiene to an existing project |

Each `SKILL.md` is the canonical source — read those for the rules. The bootstrap stays lightweight so it can ride in every session without burning context.

## Quickstart

Installation differs by harness. Pick the section for the agent you use.

### Claude Code

The plugin lives in this repo. To register it as a development marketplace:

```bash
/plugin marketplace add gg-mo/repo-hygiene
/plugin install repo-hygiene@repo-hygiene-dev
```

Or for a local checkout:

```bash
/plugin marketplace add /path/to/repo-hygiene
/plugin install repo-hygiene@repo-hygiene-dev
```

### Codex CLI / Codex App

The Codex manifest lives in `.codex-plugin/plugin.json`. Install via the plugins UI by pointing to this repo, or via:

```bash
codex plugin install https://github.com/gg-mo/repo-hygiene
```

### Cursor

```bash
cursor plugin install https://github.com/gg-mo/repo-hygiene
```

The Cursor-specific hook config is at `hooks/hooks-cursor.json`.

### OpenCode

```bash
opencode plugin install repo-hygiene
```

The OpenCode adapter is `.opencode/plugins/repo-hygiene.js`. It registers `skills/` with OpenCode's skill loader and transforms the first user message of each session to include the bootstrap.

### Gemini CLI

```bash
gemini extension install gg-mo/repo-hygiene
```

The Gemini entry file [`GEMINI.md`](GEMINI.md) imports the bootstrap skill via `@`-reference. Tool-name mappings (e.g. `TodoWrite` → equivalent) are documented inside the bootstrap.

### GitHub Copilot CLI

```bash
copilot plugin install gg-mo/repo-hygiene
```

The shared SessionStart hook auto-detects Copilot CLI via the `COPILOT_CLI=1` environment variable and emits the SDK-standard `additionalContext` JSON.

### Local install (no plugin)

If you don't want to register a plugin, drop the skills into your local Claude Code skills directory:

```bash
# User-level (applies to all your sessions)
cp -r skills/* ~/.claude/skills/

# Project-level (applies only inside this repo)
cp -r skills/* /path/to/your/project/.claude/skills/
```

You'll lose the SessionStart auto-bootstrap (so `using-repo-hygiene` won't auto-load), but the sub-skills will still be discoverable by the `Skill` tool when their descriptions match.

## Bypass

A user can suppress any hygiene skill for one turn by including `#hygiene-skip <reason>` in their message. The agent acknowledges the bypass and proceeds. This is for the cases where the skill would be genuinely wrong — not a way to disable hygiene permanently.

## How the skills were built

Each sub-skill was developed via the TDD-for-skills cycle from [`superpowers:writing-skills`](https://github.com/obra/superpowers/tree/main/skills/writing-skills):

1. **RED** — dispatch a subagent with a representative scenario, no skill loaded; document its baseline behavior verbatim.
2. **GREEN** — write the skill addressing the specific failure modes observed.
3. **REFACTOR** — re-test with the skill loaded; tighten any new rationalizations.

The baseline scenarios surfaced honest admissions like *"without this nudge I'd treat 'ship it' as authorization to commit whatever's staged"* and *"user said commit, so commit."* The skills are tuned against those exact failure modes. See [CLAUDE.md](CLAUDE.md) for how to test changes when contributing.

## Repo layout

```
.
├── .claude-plugin/         # Claude Code plugin manifest + marketplace
├── .codex-plugin/          # Codex CLI / App manifest
├── .cursor-plugin/         # Cursor manifest
├── .opencode/plugins/      # OpenCode JS adapter
├── hooks/
│   ├── hooks.json          # Claude Code SessionStart registration
│   ├── hooks-cursor.json   # Cursor SessionStart registration
│   ├── run-hook.cmd        # Polyglot batch/bash wrapper (Windows + Unix)
│   └── session-start       # Bash script that injects the bootstrap
├── skills/
│   ├── using-repo-hygiene/         # Bootstrap (loaded at session start)
│   ├── orienting-to-repo/
│   ├── writing-docstrings/
│   ├── keeping-files-small/
│   ├── keeping-docs-fresh/
│   ├── testing-new-code/
│   ├── writing-commit-messages/
│   ├── gating-commits/
│   └── cleaning-existing-codebase/
├── gemini-extension.json   # Gemini CLI extension manifest
├── GEMINI.md               # Gemini bootstrap entry (@-imports the skill)
├── CLAUDE.md               # Contributor guidelines
├── AGENTS.md               # → CLAUDE.md (symlink)
├── package.json            # OpenCode entry + npm metadata
└── README.md
```

## Contributing

See [CLAUDE.md](CLAUDE.md). Short version: skills are documentation that shapes agent behavior. Don't edit them without the RED → GREEN → REFACTOR cycle. One problem per PR.

## License

MIT. See [LICENSE](LICENSE).
