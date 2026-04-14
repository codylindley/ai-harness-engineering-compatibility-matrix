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

The table maps config files across five categories:

| Category | Examples |
|---|---|
| **Always-on instructions** | `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `copilot-instructions.md`, `*.instructions.md`, `.claude/rules/*.md`, `.cursor/rules/*.md`, `.windsurf/rules/*.md` |
| **Custom agents** | `.github/agents/*.agent.md`, `.claude/agents/*.md`, `.opencode/agents/*.md`, `.gemini/agents/*.md` |
| **Skills** | `.github/skills/*/SKILL.md`, `.claude/skills/*/SKILL.md`, `.agents/skills/*/SKILL.md`, `.gemini/skills/*/SKILL.md`, `.opencode/skills/*/SKILL.md`, `.cursor/skills/*/SKILL.md`, `.windsurf/skills/*/SKILL.md` |
| **Prompt files** | `*.prompt.md`, `.claude/commands/*.md`, `.opencode/commands/*.md`, `.gemini/commands/*.toml`, `.windsurf/workflows/*.md` |
| **Lifecycle hooks** | `.github/hooks/`, `.claude/hooks/`, `.codex/hooks.json`, `.windsurf/hooks.json` |

Each cell indicates support level: **Native**, **Fallback/compat**, **Opt-in/partial**, or **Not supported**.

## Key takeaways

- `AGENTS.md` has the broadest reach of any instructions file in the matrix — natively read by Copilot, Codex, OpenCode, Cursor, Windsurf, and Amp, with opt-in support from Gemini CLI. Claude Code still relies on `CLAUDE.md`; Amp falls back to it when no `AGENTS.md` exists.
- `.agents/skills/` is now the broadest cross-tool skill path — read natively by Codex, Cursor, and Amp, with fallback support from OpenCode, Windsurf, and Gemini CLI.
- Cursor and Windsurf both support four-mode scoped rules (`.cursor/rules/` and `.windsurf/rules/`) with always-on, glob, model-decision, and manual activation.
- Windsurf has the most complete hooks system: 12 event types with pre/post triggers and three-tier config merging (system → user → workspace).
- MCP server config remains fully fragmented — every tool uses a different file and format. Windsurf’s MCP config is notably user-level only.

## Features

- Light/dark mode via `prefers-color-scheme`
- Expandable info rows (tap the **ⓘ** icon on any row)
- Footnoted edge cases for partial support
- No dependencies — pure HTML/CSS/JS

## Last updated

April 2026
