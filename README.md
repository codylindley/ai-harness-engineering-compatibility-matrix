# AI Harness Engineering Compatibility Matrix

A single-page reference table showing which AI coding tool configuration files are supported by which tools — covering **GitHub Copilot**, **Claude Code**, **OpenCode**, **Codex CLI**, **Gemini CLI**, **Cursor**, **Windsurf**, and **Amp**.

**Live page:** [codylindley.github.io/ai-harness-engineering-compatibility-matrix](https://codylindley.github.io/ai-harness-engineering-compatibility-matrix/)

## Wait, what kind of engineering is this now?

The AI industry has a proud tradition of slapping "engineering" onto things until they sound important enough to put on a résumé.

**Prompt engineering** (2022–2023) — The OG. You learned that saying "please" to GPT-3 actually helped, that "think step by step" was basically a cheat code, and that your job title could now include "engineer" even if your main tool was a textarea. What you ask.

**Context engineering** (2024–2025) — Turns out the model is only as good as the *context* you feed it. So now you're meticulously curating the entire context window — retrieval pipelines, system prompts, few-shot examples, tool outputs. It's less "talking to the AI" and more "assembling the AI's entire worldview before it wakes up." What you send.

**Harness engineering** (2025–present) — The final boss. Mitchell Hashimoto coined the term in early February 2026: every time the agent makes a mistake, don't just hope it does better next time — engineer the environment so it can't make that specific mistake the same way again. The harness is everything the model touches that *isn't* the model itself: tools, permissions, state management, verification loops, evals, checkpoints, sub-agents, observability, session handoff artifacts, hooks, and guardrails. `AGENTS.md` files, scoped instructions, skills folders, lifecycle hooks, MCP configurations — all harness. You version them, review them in PRs, and refactor them when they drift. Birgitta Böckeler at Thoughtworks breaks it into two control types: *guides* (feedforward controls that steer before the agent acts) and *sensors* (feedback controls that observe after and enable self-correction). How the whole system operates around the model.

This compatibility matrix exists because harness engineering has produced roughly 47 different config file formats across 4 tools, and someone had to make a spreadsheet. You're welcome.

## What's in the matrix

The table maps config files across eight categories. GitHub Copilot is split into three surfaces (VS Code, CLI, Cloud agent) for ten tool columns total.

| Category | Type | Examples |
|---|---|---|
| **Repo-wide instructions** | Guide | `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `copilot-instructions.md` |
| **Path-scoped instructions / rules** | Guide | `*.instructions.md`, `.claude/rules/*.md`, `.cursor/rules/*.mdc`, `.windsurf/rules/*.md` |
| **Custom / sub agents** | Guide | `.github/agents/*.agent.md`, `.claude/agents/*.md`, `.opencode/agents/*.md`, `.gemini/agents/*.md` |
| **Skills** | Guide | `.github/skills/*/SKILL.md`, `.claude/skills/*/SKILL.md`, `.agents/skills/*/SKILL.md`, `.gemini/skills/*/SKILL.md`, `.opencode/skills/*/SKILL.md`, `.cursor/skills/*/SKILL.md`, `.windsurf/skills/*/SKILL.md` |
| **Prompt files / slash commands** | Guide | `*.prompt.md`, `.claude/commands/*.md`, `.opencode/commands/*.md`, `.gemini/commands/*.toml`, `.windsurf/workflows/*.md` |
| **Automation (lifecycle hooks)** | Sensor | `.github/hooks/`, `.claude/settings.json`, `.codex/hooks.json`, `.windsurf/hooks.json` |
| **MCP server configuration** | Guide | `.vscode/mcp.json`, `.mcp.json`, `opencode.json`, `.codex/config.toml`, `.gemini/settings.json`, `.cursor/mcp.json`, `mcp_config.json`, `.amp/settings.json` |
| **Permissions / sandboxing** | Sensor | `permissions` blocks in `.claude/settings.json`, `opencode.json`, `.amp/settings.json`; `~/.copilot/permissions-config.json`; Codex `approval_policy` + `sandbox_mode`; Gemini `GEMINI_SANDBOX`; Cursor sandbox + `.cursor/cli.json` |

Each cell indicates support level: **Native**, **Fallback/compat**, **Opt-in/partial**, or **Not supported**. Category types follow Birgitta Böckeler's taxonomy: **Guide** = feedforward control (steers before the agent acts), **Sensor** = feedback control (observes after the agent acts).

**Trust boundary:** all rows are committed (team-shared, version-controlled) by default. Exceptions are visually flagged in the matrix — `user-local` (per-user, doesn't travel with the repo) for Windsurf's `mcp_config.json`, and `both` for the Codex and Windsurf hooks files which load from project and user scopes simultaneously.

**Precedence & conflicts:** a section below the matrix documents per-tool load order, settings layering, and merge-vs-override semantics — what wins when files collide.

## Key takeaways

- `AGENTS.md` has the broadest reach of any instructions file in the matrix — natively read by Copilot, Codex, OpenCode, Cursor, Windsurf, and Amp, with opt-in support from Gemini CLI. Claude Code still relies on `CLAUDE.md`; Amp falls back to it when no `AGENTS.md` exists.
- `.agents/skills/` is now the broadest cross-tool skill path — read natively by Copilot, Codex, Cursor, Amp, and Gemini CLI, with fallback support from OpenCode and Windsurf.
- Cursor and Windsurf both mirror the four-mode scoped rules pattern (`.cursor/rules/` and `.windsurf/rules/`) with repo-wide, glob, model-decision, and manual activation — the most granular path-scoped controls in the matrix.
- Claude Code has the most complete hooks system among the tools shown here, with core lifecycle/tool-use events and five handler types in the agent SDK. Windsurf follows with 12 events and three-tier config merging. Copilot hooks are documented for CLI and cloud-agent workflows, and available in Preview for VS Code (which also reads Claude-format `.claude/settings.json` hooks).
- MCP server config remains fully fragmented — every tool uses a different file and format. Windsurf's MCP config is notably user-level only with no project-scoped committable file.
- **Codex, Gemini CLI, and Cursor are the only tools with kernel-enforced OS sandboxes** (Seatbelt / Landlock / Docker). Copilot Cloud Agent gets isolation for free via the ephemeral GitHub Actions VM. Every other tool relies on permission rules alone — meaning a misbehaving agent with the right rule match can still touch anything the user can.

## What happens when multiple instruction files coexist

Most repos will end up with more than one instruction file — `AGENTS.md` alongside `copilot-instructions.md`, or `CLAUDE.md` in both root and subdirectories. Precedence behavior varies by tool:

| Scenario | Behavior |
|---|---|
| `AGENTS.md` + `copilot-instructions.md` | Copilot loads both additively — no override, no conflict. Use `copilot-instructions.md` for Copilot-specific additions. |
| `AGENTS.md` + `CLAUDE.md` (same repo) | Copilot reads both. Claude Code reads only `CLAUDE.md` (ignores `AGENTS.md`). Use `@AGENTS.md` in your `CLAUDE.md` to import shared rules. |
| `CLAUDE.md` in root + subdirectory | Claude Code merges hierarchically — subdirectory rules layer on top of root rules for files in that directory. |
| `GEMINI.md` at global + project + subdirectory | Gemini CLI merges all three tiers with just-in-time discovery — narrower scope layers on top of broader. |
| `AGENTS.md` + `CLAUDE.md` in OpenCode | OpenCode prefers `AGENTS.md` in a directory and only falls back to `CLAUDE.md` when `AGENTS.md` is absent. |
| Multiple `.cursor/rules/*.mdc` files | All matching rules are merged. `alwaysApply` rules load every session; glob/model/manual rules are additive when triggered. |
| Path-scoped rules + repo-wide instructions | Usually additive in tools that support both, but tool-specific precedence still matters. Check the matrix notes before assuming both files load. |

The general principle: **many instruction systems are additive, but not all are.** Some tools merge every matching source; others use fallback or nearest-file precedence. When in doubt, keep repo-wide files short and use scoped rules for specifics.

## Features

- Light/dark mode with manual toggle (defaults to `prefers-color-scheme`)
- Column toggle filters — show/hide any tool to focus comparisons
- Expandable info rows (tap the **ⓘ** icon on any row)
- Clickable category headers open detail modals with examples and comparison tabs
- GitHub star count badge (live from API)
- No framework dependencies — pure HTML/CSS/JS

## Last updated

April 2026
