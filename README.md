# AI Harness Engineering Compatibility Matrix

A single-page reference for which AI coding harness configuration files are supported where, covering **GitHub Copilot**, **Claude Code**, **OpenCode**, **Codex**, **Gemini CLI**, **Cursor**, **Devin Desktop**, **Amp**, and **JetBrains Junie**.

**Live page:** [codylindley.github.io/ai-harness-engineering-compatibility-matrix](https://codylindley.github.io/ai-harness-engineering-compatibility-matrix/)

## What Kind Of Engineering Is This?

These terms describe different layers of control around AI coding tools:

**Prompt engineering** shapes the request: what you ask the model to do.

**Context engineering** shapes the material available to the model: system prompts, retrieved docs, examples, tool output, and other context.

**Harness engineering** shapes the system around the model: tools, permissions, state, checks, hooks, agents, MCP servers, and guardrails. [Mitchell Hashimoto coined the term](https://mitchellh.com/writing/my-ai-adoption-journey#step-5-engineer-the-harness) in early February 2026 for the work of fixing agent mistakes by changing the environment, not just the prompt.

This matrix focuses on harness files you can version, review, and refactor: `AGENTS.md`, scoped rules, skills, prompt files, lifecycle hooks, MCP config, and permission/sandbox settings. It uses [Birgitta Böckeler's control taxonomy](https://martinfowler.com/articles/harness-engineering.html): **Guide** means feedforward context that steers the agent before it acts; **Sensor** means feedback or enforcement that observes, blocks, or corrects after the agent acts.

## What's In The Matrix

The table maps config files across eight harness categories. GitHub Copilot is split into VS Code, CLI/app, and cloud-agent surfaces, for eleven product columns total.

| Category | Type | Examples |
|---|---|---|
| **Repo-wide instructions** | Guide | `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `copilot-instructions.md` |
| **Path-scoped instructions / rules** | Guide | `*.instructions.md`, `.claude/rules/*.md`, `.cursor/rules/*.mdc`, `.devin/rules/*.md`, `.junie/rules/*.md` |
| **Custom / sub agents** | Guide | Markdown agent profiles, Codex `[agents]`, Cursor modes, Amp plugins |
| **Skills** | Guide | Tool-native `SKILL.md` folders and the shared `.agents/skills/` path |
| **Prompt files / slash commands** | Guide | `*.prompt.md`, command folders, Gemini TOML commands, Cascade workflows |
| **Automation (lifecycle hooks)** | Sensor | Copilot, Claude, Codex, Gemini, Cursor, Devin, Amp, and Junie hook systems |
| **MCP server configuration** | Guide | Project and user MCP files for all nine tool families |
| **Permissions / sandboxing** | Sensor | Permission policies, approval modes, and OS-enforced sandboxes |

Each cell indicates support level: **Native**, **Fallback/compat**, **Opt-in/partial**, or **Not supported**.

**Trust boundary:** rows are committed, team-shared, and version-controlled unless flagged otherwise. `user-local` means per-user and not carried with the repo. `both` means the tool loads project and user scopes at the same time.

**Precedence & conflicts:** the page includes per-tool load order, settings layering, and merge-vs-override behavior so you can see what wins when files collide.

## Key takeaways

- `AGENTS.md` has the broadest reach of any instruction file here. Copilot, OpenCode, Codex, Cursor, Devin Desktop, Amp, and Junie read it natively; Gemini CLI can opt in. Claude Code still uses `CLAUDE.md`.
- `.agents/skills/` is the broadest shared skill path. Seven tool families read it natively, and OpenCode supports it as a compatibility location.
- Cursor and Devin Desktop provide the most granular scoped rules: repo-wide, glob, model-decision, and manual activation.
- Claude Code has the richest hook system. Copilot, Codex, Gemini CLI, Cursor, Devin Desktop, and Amp also support hooks; Junie hooks are in EAP.
- MCP configuration is still fragmented. Devin Local now supports committed `.devin/mcp_config.json`; only legacy Cascade remains user-local.
- Copilot CLI and Devin Local added opt-in OS sandboxes, joining Claude Code, Codex, Gemini CLI, and Cursor.

## When Multiple Instruction Files Coexist

Most repos will have more than one instruction file, such as `AGENTS.md` plus `copilot-instructions.md`, or `CLAUDE.md` in both root and subdirectories. Some tools merge matching files; others use fallback or nearest-file precedence.

| Scenario | Behavior |
|---|---|
| `AGENTS.md` + `copilot-instructions.md` | Copilot loads both additively — no override, no conflict. Use `copilot-instructions.md` for Copilot-specific additions. |
| `AGENTS.md` + `CLAUDE.md` (same repo) | Copilot reads both. Claude Code reads only `CLAUDE.md`; use `@AGENTS.md` inside it to import shared rules. |
| `CLAUDE.md` in root + subdirectory | Claude Code merges hierarchically — subdirectory rules layer on top of root rules for files in that directory. |
| `GEMINI.md` at global + project + subdirectory | Gemini CLI merges all three tiers with just-in-time discovery — narrower scope layers on top of broader. |
| `AGENTS.md` + `CLAUDE.md` in OpenCode | OpenCode prefers `AGENTS.md` in a directory and only falls back to `CLAUDE.md` when `AGENTS.md` is absent. |
| Multiple `.cursor/rules/*.mdc` files | All matching rules are merged. `alwaysApply` rules load every session; glob/model/manual rules are additive when triggered. |
| Path-scoped rules + repo-wide instructions | Usually additive in tools that support both, but tool-specific precedence still matters. Check the matrix notes before assuming both files load. |

The practical rule: keep repo-wide files short, use scoped rules for specifics, and check the tool's precedence before assuming two files both load.

## Features

- Light/dark mode with manual toggle (defaults to `prefers-color-scheme`)
- Accessible column filters for focused comparisons
- Expandable, keyboard-operable row details
- Category dialogs with focus management, examples, and comparison tabs
- Sticky config labels inside the horizontally scrollable mobile table
- Primary-source links grouped by product
- GitHub star count badge (live from API)
- No framework dependencies — pure HTML/CSS/JS

## Last updated

August 23, 2026
