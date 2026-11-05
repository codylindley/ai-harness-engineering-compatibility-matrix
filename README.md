# AI Harness Engineering Compatibility Matrix

A single-page reference for which AI coding harness configuration files are supported across **GitHub Copilot**, **Claude Code**, **OpenAI Codex**, and **Cursor**.

**Live page:** [codylindley.github.io/ai-harness-engineering-compatibility-matrix](https://codylindley.github.io/ai-harness-engineering-compatibility-matrix/)

## What Kind Of Engineering Is This?

These terms describe different layers of control around AI coding tools:

**Prompt engineering** shapes the request: what you ask the model to do.

**Context engineering** shapes the material available to the model: system prompts, retrieved docs, examples, tool output, and other context.

**Harness engineering** shapes the system around the model: tools, permissions, state, checks, hooks, agents, MCP servers, and guardrails. [Mitchell Hashimoto coined the term](https://mitchellh.com/writing/my-ai-adoption-journey#step-5-engineer-the-harness) in early February 2026 for the work of fixing agent mistakes by changing the environment, not just the prompt.

This matrix focuses on harness files you can version, review, and refactor: `AGENTS.md`, scoped rules, skills, prompt files, lifecycle hooks, MCP config, and permission/sandbox settings. It uses [Birgitta Böckeler's control taxonomy](https://martinfowler.com/articles/harness-engineering.html): **Guide** means feedforward context that steers the agent before it acts; **Sensor** means feedback or enforcement that observes, blocks, or corrects after the agent acts.

## What's In The Matrix

The table maps config files across nine harness categories. GitHub Copilot is split into the VS Code **Copilot harness**, desktop app, CLI, and cloud-agent surfaces because those environments expose materially different configuration paths. Claude Code and OpenAI Codex each use one **local surfaces** column because their desktop, CLI, and editor clients share the same core configuration layers. Cursor remains combined across its editor and CLI. Claude Code, Codex, and Cursor cloud runtimes are outside those local columns unless a note explicitly calls them out.

This follows the product/surface framing of [Slash Command Atlas](https://codylindley.github.io/slash-command-atlas/) without copying its granularity wholesale: slash-command menus diverge by interface more often than harness configuration does.

| Category | Type | Examples |
|---|---|---|
| **Repo-wide instructions** | Guide | `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `copilot-instructions.md`, `github-app.yml` |
| **Path-scoped instructions / rules** | Guide | `*.instructions.md`, `.claude/rules/*.md`, `.cursor/rules/*.mdc` |
| **Custom / sub agents** | Guide | Markdown agent profiles for Copilot, Claude Code, and Cursor; Codex TOML agents |
| **Skills** | Guide | Tool-native `SKILL.md` folders and the shared `.agents/skills/` path |
| **Prompt files / legacy commands** | Guide | Claude command files; VS Code Local prompt files are outside the Copilot-harness column |
| **Automation (lifecycle hooks)** | Sensor | GitHub Copilot, Claude Code, OpenAI Codex, and Cursor hook systems |
| **MCP server configuration** | Guide | Project and user MCP files for the four product families |
| **Permissions / sandboxing** | Sensor | Permission policies, approval modes, and OS-enforced sandboxes |
| **Packaging & environment** | Guide | Plugin manifests, marketplaces, and cloud-agent provisioning |

Each cell indicates support level: **Native**, **Fallback/compat**, **Opt-in/partial**, or **Not supported**.

**Trust boundary:** rows are committed, team-shared, and version-controlled unless flagged otherwise. `user-local` means per-user and not carried with the repo. `both` means the tool loads project and user scopes at the same time. Above all of these sits an org-managed tier — `managed-settings.json`, Codex `requirements.toml`, enforced Cursor Team Rules — that users cannot override.

**Precedence & conflicts:** the page includes per-tool load order, settings layering, and merge-vs-override behavior so you can see what wins when files collide.

## Key takeaways

- `AGENTS.md` is now the portable instruction baseline across all four product families. GitHub Copilot, OpenAI Codex, and Cursor read it directly. Claude Code v2.1.277+ reads it when no `CLAUDE.md`, `.claude/CLAUDE.md`, or `CLAUDE.local.md` exists in or above the working directory, and can be configured to load both formats.
- `.agents/skills/` is the broadest shared skill path across the selected products: GitHub Copilot, OpenAI Codex, and Cursor read it. Claude Code uses `.claude/skills/`, but its skills follow the same open [Agent Skills](https://agentskills.io/) format.
- Root `plugin.json` with the Agent Plugins schema is the portable bundle baseline for GitHub Copilot, OpenAI Codex, and Cursor. The standard packages skills and `mcp.json`; vendor-only agents, hooks, rules, and UI extensions stay in vendor namespaces or vendor manifests. Copilot also accepts a legacy schema-less `plugin.json`, and Cursor uses its own plugin-root placeholder inside portable MCP configuration.
- Compatibility still matters, but a shared directory is not automatically a shared file format. Copilot CLI reads Claude rules and agents, Cursor imports Claude hooks and Claude/Codex agent or skill locations, and Codex agent TOML remains different from Cursor-compatible Markdown.
- Surface splitting is useful only when the harness changes. GitHub Copilot stays split, and the VS Code column means the Copilot harness rather than every harness VS Code can host. Claude Code and OpenAI Codex local clients stay aggregated.
- Claude Code has the richest hook system at roughly 33 documented events. Its `prompt`/`agent` handlers and Cursor's prompt hooks are inferential sensors; current Codex hooks execute commands or MCP tools instead.
- `.mcp.json` is shared by the VS Code Copilot harness, GitHub Copilot app, GitHub Copilot CLI, and Claude Code. VS Code can also forward eligible servers from `.vscode/mcp.json`; `.github/mcp.json`, `.codex/config.toml`, and `.cursor/mcp.json` remain product-specific.
- Every local coding-agent family shown here provides OS-enforced isolation. GitHub Copilot local sandboxing is now generally available in the app, CLI, and VS Code Agent Host sessions.
- An org-managed tier still overrides repository and user configuration. Bundles make that governance more important because a plugin can carry executable hooks and MCP servers alongside guidance.

## When Multiple Instruction Files Coexist

Most repos will have more than one instruction file, such as `AGENTS.md` plus `copilot-instructions.md`, or `CLAUDE.md` in both root and subdirectories. Some tools merge matching files; others use fallback or nearest-file precedence.

| Scenario | Behavior |
|---|---|
| `AGENTS.md` + `copilot-instructions.md` | Copilot combines both when they apply, but does not define a general precedence order. Keep shared rules in `AGENTS.md` and Copilot-specific additions in `copilot-instructions.md`. |
| `AGENTS.md` + `CLAUDE.md` (same repo) | Claude Code reads only `CLAUDE.md` by default, can be configured to load both, or can import `AGENTS.md` with `@AGENTS.md`. Copilot CLI and cloud agent load both; the VS Code Copilot harness loads `AGENTS.md`, while VS Code's Claude or Local harness handles `CLAUDE.md`. |
| `CLAUDE.md` in root + subdirectory | Claude Code merges hierarchically — subdirectory rules layer on top of root rules for files in that directory. |
| Multiple `.cursor/rules/*.mdc` files | All matching rules are merged. `alwaysApply` rules load every session; glob/model/manual rules are additive when triggered. |
| Path-scoped rules + repo-wide instructions | Usually additive in tools that support both, but tool-specific precedence still matters. Check the matrix notes before assuming both files load. |

The practical rule: keep repo-wide files short, use scoped rules for specifics, and check the tool's precedence before assuming two files both load.

## Features

- Light/dark mode with manual toggle (defaults to `prefers-color-scheme`)
- Accessible column filters for focused comparisons — all seven surfaces are shown by default
- Expandable, keyboard-operable row details
- Category dialogs with focus management, examples, and comparison tabs — open one by clicking any category heading row
- Sticky config labels inside the horizontally scrollable mobile table
- Primary-source links grouped by product
- GitHub star count badge (live from API)
- No framework dependencies — pure HTML/CSS/JS

## Last updated

November 5, 2026
