---
summary: "Repo structure, platform concepts (routing, skills, ACPX), and editing checklists"
read_when:
  - "Understanding how the kit is organized"
  - "Adding a template, doc, skill, or config key"
  - "Changing config schema or the example files"
---

# Architecture

**OpenClaw Multi-Agent Kit** — Templates and docs for building AI agent teams on [OpenClaw](https://openclaw.sh) with Telegram supergroup integration. This is a **template/docs-only repo** (no runtime code). It provides SOUL.md personality templates, IDENTITY.md metadata, workspace scaffolding, skill packages, and `openclaw.json` config snippets.

The kit ships **two layers of soul templates**:
- A small set of **core templates** in `templates/soul/*.md` (orchestrator, coder, QA, devops, research, growth, content, community, leadgen, ops) — the production-tested 10-agent starter team.
- An **extended catalog** in `templates/soul/<category>/*.md` (106 files across 12 categories: design, engineering, game-development, marketing, performance-marketing, product, project-management, sales, spatial-computing, specialized, support, testing) — adapted from agency-style role libraries. These are pre-adaptation starters: usable, but the body content is generic AI-agent prose, not yet rewritten with OpenClaw-specific routing/handoff details. They each carry an "OpenClaw Adaptation Notes" footer pointing to what still needs to be wired up.

**Not a library or app.** Nothing here executes — everything is markdown, JSON, and JSONC meant to be copied into an OpenClaw workspace.

## Tech stack / platform

- **Platform:** OpenClaw (agent orchestration platform)
- **Channel:** Telegram (supergroups with topics, plus DM forum topics)
- **LLM providers:** Provider-agnostic — templates do not pin a model. Agents inherit whatever model the OpenClaw runtime is configured with. Per-agent `model` overrides are documented but intentionally absent from example configs.
- **Config format:** JSON / JSONC
- **Docs/templates:** Markdown

## Repository structure

```
.
├── AGENTS.md                          # Agent hard rules (read first)
├── CLAUDE.md                          # Pointer to AGENTS.md
├── CHANGELOG.md                       # Keep-a-Changelog
├── INSTRUCTIONS.md                    # AI-readable setup guide (8 phases)
├── README.md                          # Project overview & quick start
├── LICENSE                            # MIT
├── .agents/commands/                  # Cross-agent slash commands (handoff/pickup/commit/fix/changelog)
├── docs/
│   ├── index.md                       # Docs index — routing data for agents
│   ├── architecture.md                # This file
│   ├── acpx-telegram.md               # ACPX coding agent backend for Telegram
│   ├── advanced-openclaw-practices.md # 13 high-leverage operating patterns
│   ├── agent-design-patterns.md       # How to write effective SOUL.md files
│   ├── inter-agent-handoff-standard.md# HANDOFF/ACK/DONE/BLOCKED contract for sessions_send
│   ├── scaling.md                     # When to add agents, cost tiers, circular triggers
│   ├── skills-system.md               # Native skills system, ClawHub, custom skills
│   ├── supergroup-setup.md            # Step-by-step Telegram supergroup setup
│   ├── telegram-channel-architecture.md # Production lane design for topic routing
│   └── telegram-dm-topics.md          # Telegram DM forum topics + ACP binding
├── examples/
│   ├── full-team.json                 # Complete 10-agent openclaw.json config (no model pinned)
│   └── minimal-team.json              # Minimal 3-agent config (orchestrator + coder + QA)
└── templates/
    ├── openclaw-config.jsonc          # Base config snippet with defaults (model-agnostic)
    ├── identity/agent-identity.md     # Standard identity template for any agent
    ├── soul/                          # Agent personality templates (SOUL.md)
    │   ├── README.md                  # Catalog overview + adaptation checklist
    │   ├── <10 core agent templates>  # orchestrator, coding, qa, devops, research, growth, content, community, leadgen, ops
    │   └── <12 category dirs>         # Extended catalog (106 files)
    ├── workspace/                     # Shared context file templates
    │   ├── AGENTS.md                  # Orchestrator operations guide
    │   ├── FEEDBACK-LOG.md            # Style corrections and lessons
    │   ├── SIGNALS.md                 # Shared intelligence hub
    │   ├── SUPERGROUP-MAP.md          # Topic and agent mapping
    │   └── THESIS.md                  # Business thesis — north star for all agents
    └── skills/                        # Skill templates (SKILL.md)
        ├── README.md                  # Skill catalog + path convention
        └── <8 skill dirs>             # coding-handoff, research-intel, leadgen-qualification,
                                       # content-repurpose, ops-triage, telegram-topic-setup,
                                       # acpx-session, template-audit
```

## Key concepts

- **Three Telegram routing models** — Multi-bot routing, native topic routing, and DM forum topics (see docs/)
- **One topic per team** — Teams share a topic channel in a supergroup
- **Primary + Secondary agents** — Primary owns the topic; secondary responds only when @mentioned or triggered
- **Shared context via markdown files** — Agents coordinate through THESIS.md, SIGNALS.md, FEEDBACK-LOG.md (not APIs)
- **Bot-to-bot via `sessions_send`** — Telegram bots cannot see each other's messages; OpenClaw's `sessions_send` bridges them
- **Structured handoffs** — All cross-agent requests follow the HANDOFF/ACK/DONE/BLOCKED contract in `docs/inter-agent-handoff-standard.md`
- **Structured escalation** — Agents escalate to orchestrator; orchestrator escalates to human

### Telegram routing models

| Model | Visibility | Best For |
|-------|-----------|----------|
| **Multi-bot routing** | Each agent has its own bot identity | Specialist teams with visible personas |
| **Native topic routing** | One bot, different internal agents per topic | Clean single-bot UX with internal specialization |
| **DM forum topics** | Topics inside a direct chat | Private 1:1 organized conversations with ACP support |

### Skills system (v2026.3.24+)

OpenClaw includes a native Skills system with one-click install from ClawHub, Control UI management, and CLI tools. Skills are placed at `agents/<agent>/skills/<skill-name>/SKILL.md` and use YAML frontmatter + markdown body format.

### ACPX coding subagents

Agents can delegate coding work to dedicated coding agents (Claude Code, Codex, OpenCode, Gemini CLI, Pi, Copilot, Cursor, Droid, Kimi, Kiro, Qwen, Trae) via ACPX. Define a coder agent with `runtime.type: "acp"` in openclaw.json. Other agents trigger it via `sessions_spawn(runtime="acp")` from a subagent session. Thread bindings (`channels.telegram.threadBindings.spawnSessions: true`, optionally per-account) create persistent sessions per topic. Canonical OpenClaw docs live at **`docs.openclaw.ai`** — verify config keys against the live reference before editing schema-related templates.

Known platform issues to remember when designing templates:
1. `sessions_send` between agents can corrupt the target session's channel field from `telegram` to `webchat` ([openclaw/openclaw#31671](https://github.com/openclaw/openclaw/issues/31671)) — receivers should post lifecycle messages explicitly via the `message` tool, not rely on the target's channel state.
2. `sessions_spawn(runtime="acp", thread:true)` is not supported for Telegram ([openclaw/openclaw#41004](https://github.com/openclaw/openclaw/issues/41004)) — use `/acp spawn --thread here` or a persistent ACP binding instead. On Telegram, the `/acp spawn` flag is `--thread here|auto`, not `--bind here` (`--bind here` is for Discord/BlueBubbles/iMessage only).

### Team layout (default)

| Team     | Topic    | Primary Agent | Secondary Agents    |
|----------|----------|---------------|---------------------|
| General  | Topic 1  | Orchestrator  | —                   |
| Build    | Topic N  | Coder         | QA, DevOps          |
| Research | Topic N  | Researcher    | Growth              |
| Social   | Topic N  | Content       | Community           |
| Leads    | Topic N  | Lead Gen      | —                   |
| Ops      | Topic N  | Ops           | —                   |

The coder agent can optionally use ACPX (`runtime.type: "acp"`) to delegate to coding subagents. Add an ACP binding in the Build topic for persistent coding sessions.

### Critical config rule: `requireMention`

- **Multi-agent topics:** ALL bots must have `requireMention: true` — otherwise one bot responds to everything
- **Single-agent topics:** The sole agent can use `requireMention: false`
- **Orchestrator:** Must have `enabled: false` on topics owned by other agents

## Editing guidelines

- When adding a new **core** agent template: add the soul template in `templates/soul/`, update `README.md` tables, update `examples/full-team.json`, update `templates/soul/README.md` core starters list, and update the structure tree in this file.
- When adding a new **extended** soul template: drop it in the right category under `templates/soul/<category>/`, ensure YAML frontmatter is at line 1, and bump the file count in this file.
- When adding a new workspace template: add in `templates/workspace/`, update `README.md`, and update this file.
- When adding a new doc: drop it in `docs/` with `summary:`/`read_when:` frontmatter, link it from `README.md`, add it to `docs/index.md`, and update the tree here.
- When adding a new skill: create `templates/skills/<skill-name>/SKILL.md`, update `templates/skills/README.md` and the tree here.
- When changing config schema or keys: update `templates/openclaw-config.jsonc`, both example files, and `INSTRUCTIONS.md`.
- All markdown templates use `---` horizontal rules as section separators (separate from YAML frontmatter fences).
