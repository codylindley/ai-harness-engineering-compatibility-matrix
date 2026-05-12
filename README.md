# AI Harness Engineering Compatibility Matrix

A single-page reference for which AI coding tool configuration files are supported by which tools, covering **GitHub Copilot**, **Claude Code**, **OpenCode**, **Codex CLI**, **Gemini CLI**, **Cursor**, **Windsurf**, and **Amp**.

**Live page:** [codylindley.github.io/ai-harness-engineering-compatibility-matrix](https://codylindley.github.io/ai-harness-engineering-compatibility-matrix/)

## What Kind Of Engineering Is This?

These terms describe different layers of control around AI coding tools:

**Prompt engineering** shapes the request: what you ask the model to do.

**Context engineering** shapes the material available to the model: system prompts, retrieved docs, examples, tool output, and other context.

**Harness engineering** shapes the system around the model: tools, permissions, state, checks, hooks, agents, MCP servers, and guardrails. Mitchell Hashimoto coined the term in early February 2026 for the work of fixing agent mistakes by changing the environment, not just the prompt.

This matrix focuses on harness files you can version, review, and refactor: `AGENTS.md`, scoped rules, skills, prompt files, lifecycle hooks, MCP config, and permission/sandbox settings. It uses Birgitta Böckeler's control taxonomy: **Guide** means feedforward context that steers the agent before it acts; **Sensor** means feedback or enforcement that observes, blocks, or corrects after the agent acts.

## What's In The Matrix

The table maps config files across eight harness categories. GitHub Copilot is split into VS Code, CLI, and Cloud agent surfaces, for ten tool columns total.

| Category | Type | Examples |
|---|---|---|
| **Repo-wide instructions** | Guide | `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `copilot-instructions.md` |
| **Path-scoped instructions / rules** | Guide | `*.instructions.md`, `.claude/rules/*.md`, `.cursor/rules/*.mdc`, `.windsurf/rules/*.md` |
| **Custom / sub agents** | Guide | `.github/agents/*.agent.md`, `.claude/agents/*.md`, `.opencode/agents/*.md`, `.gemini/agents/*.md` |
| **Skills** | Guide | `.github/skills/*/SKILL.md`, `.claude/skills/*/SKILL.md`, `.agents/skills/*/SKILL.md`, `.gemini/skills/*/SKILL.md`, `.opencode/skills/*/SKILL.md`, `.cursor/skills/*/SKILL.md`, `.windsurf/skills/*/SKILL.md` |
| **Prompt files / slash commands** | Guide | `*.prompt.md`, `.claude/commands/*.md`, `.opencode/commands/*.md`, `.gemini/commands/*.toml`, `.windsurf/workflows/*.md` |
| **Automation (lifecycle hooks)** | Sensor | `.github/hooks/`, `.claude/settings.json`, `.codex/hooks.json`, `.gemini/settings.json`, `.cursor/hooks.json`, `.windsurf/hooks.json` |
| **MCP server configuration** | Guide | `.vscode/mcp.json`, `.mcp.json`, `opencode.json`, `.codex/config.toml`, `.gemini/settings.json`, `.cursor/mcp.json`, `mcp_config.json`, `.amp/settings.json` |
| **Permissions / sandboxing** | Sensor | `permissions` blocks in `.claude/settings.json`, `opencode.json`, `.amp/settings.json`; `~/.copilot/permissions-config.json`; Codex `approval_policy` + `sandbox_mode`; Gemini `GEMINI_SANDBOX`; Cursor sandbox + `.cursor/cli.json` |

Each cell indicates support level: **Native**, **Fallback/compat**, **Opt-in/partial**, or **Not supported**.

**Trust boundary:** rows are committed, team-shared, and version-controlled unless flagged otherwise. `user-local` means per-user and not carried with the repo. `both` means the tool loads project and user scopes at the same time.

**Precedence & conflicts:** the page includes per-tool load order, settings layering, and merge-vs-override behavior so you can see what wins when files collide.

## Key takeaways

- `AGENTS.md` has the broadest reach of any instruction file here. Copilot, Codex, OpenCode, Cursor, Windsurf, and Amp read it natively; Gemini CLI can opt in. Claude Code still uses `CLAUDE.md`.
- `.agents/skills/` is the broadest shared skill path. Copilot, Codex, Cursor, Amp, and Gemini CLI read it natively; OpenCode and Windsurf support it as fallback.
- Cursor and Windsurf provide the most granular scoped rules: repo-wide, glob, model-decision, and manual activation.
- Claude Code has the richest hook system. Cursor, Windsurf, Gemini CLI, Codex, and Copilot also support hooks, but with different event models and config scopes.
- MCP configuration is still fragmented. Every tool uses a different file and format, and Windsurf's MCP config is user-local only.
- Claude Code, Codex, Gemini CLI, and Cursor have local OS sandboxes. Copilot Cloud Agent isolates work in an ephemeral GitHub Actions VM. The remaining local tools rely on permission rules without kernel-enforced isolation.

## When Multiple Instruction Files Coexist

Most repos will have more than one instruction file, such as `AGENTS.md` plus `copilot-instructions.md`, or `CLAUDE.md` in both root and subdirectories. Some tools merge matching files; others use fallback or nearest-file precedence.

| Scenario | Behavior |
|---|---|
| `AGENTS.md` + `copilot-instructions.md` | Copilot loads both additively — no override, no conflict. Use `copilot-instructions.md` for Copilot-specific additions. |
| `AGENTS.md` + `CLAUDE.md` (same repo) | Copilot reads both. Claude Code reads only `CLAUDE.md` (ignores `AGENTS.md`). Use `@AGENTS.md` in your `CLAUDE.md` to import shared rules. |
| `CLAUDE.md` in root + subdirectory | Claude Code merges hierarchically — subdirectory rules layer on top of root rules for files in that directory. |
| `GEMINI.md` at global + project + subdirectory | Gemini CLI merges all three tiers with just-in-time discovery — narrower scope layers on top of broader. |
| `AGENTS.md` + `CLAUDE.md` in OpenCode | OpenCode prefers `AGENTS.md` in a directory and only falls back to `CLAUDE.md` when `AGENTS.md` is absent. |
| Multiple `.cursor/rules/*.mdc` files | All matching rules are merged. `alwaysApply` rules load every session; glob/model/manual rules are additive when triggered. |
| Path-scoped rules + repo-wide instructions | Usually additive in tools that support both, but tool-specific precedence still matters. Check the matrix notes before assuming both files load. |

The practical rule: keep repo-wide files short, use scoped rules for specifics, and check the tool's precedence before assuming two files both load.

## Features

- Light/dark mode with manual toggle (defaults to `prefers-color-scheme`)
- Column toggle filters — show/hide any tool to focus comparisons
- Expandable info rows (tap the **ⓘ** icon on any row)
- Clickable category headers open detail modals with examples and comparison tabs
- GitHub star count badge (live from API)
- No framework dependencies — pure HTML/CSS/JS

## Last updated

May 2026
