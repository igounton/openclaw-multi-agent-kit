---
summary: "OpenClaw native Skills system - ClawHub install, Control UI, custom skills"
read_when:
  - "Adding, installing, or authoring a skill"
  - "A skill isn't loading or triggering"
---

# Skills System

OpenClaw v2026.3.24 introduced a native Skills system for giving agents deterministic, reusable workflows. Skills live as `SKILL.md` files in each agent's workspace and are loaded automatically by the runtime. They complement SOUL.md — where SOUL.md defines who an agent *is*, a skill defines how it *executes* a specific workflow.

---

## How Skills Work

Each skill is a single `SKILL.md` file using YAML frontmatter + markdown body:

```yaml
---
name: my-skill
description: One-line summary of what this skill does and when to use it.
---

# Skill Name

## Instructions

Step-by-step process the agent follows when this skill is active.
```

**Path convention:**

```
agents/<agent>/skills/<skill-name>/SKILL.md
```

Skills can also include a `references/` subdirectory for supporting documents the agent should read when using the skill:

```
agents/<agent>/skills/<skill-name>/
├── SKILL.md
└── references/
    ├── checklist.md
    └── examples.md
```

**Loading:** OpenClaw reads all `SKILL.md` files under an agent's `skills/` directory at session start. The agent treats skill instructions as high-priority context — skills override general behavior when their trigger conditions match.

---

## ClawHub Marketplace

[ClawHub](https://clawhub.com) is the skill marketplace for OpenClaw. Browse community and official skills, then install them directly into an agent's workspace.

**One-click install:** From the Control UI, select an agent, browse available skills, and click install. OpenClaw downloads the skill package and places it at the correct path automatically.

Install recipes may include:
- Prerequisite skills (installed as dependencies)
- Required API keys (prompted during setup)
- Configuration values (prompted during setup)

---

## Control UI

Manage skills per agent through the OpenClaw Control UI.

**Status filter tabs:**

| Tab | Meaning |
|-----|---------|
| All | Every installed skill |
| Ready | Fully configured and active |
| Needs Setup | Installed but missing API keys or config |
| Disabled | Installed but toggled off |

Each tab shows a count badge. Clicking a skill opens a detail dialog with:
- Skill description and version
- Setup requirements (API keys, config fields)
- Toggle to enable/disable
- Uninstall option

**API key entry:** When a skill requires an external service key, the detail dialog provides an input field. Keys are stored securely in the agent's vault — never in the SKILL.md file or workspace directory.

---

## CLI Management

Use the CLI for scripted or headless skill management.

```bash
openclaw skills list
```

Lists all skills across all agents with status (Ready / Needs Setup / Disabled).

```bash
openclaw skills info <skill-name>
```

Shows detailed info for a specific skill: description, version, prerequisites, required API keys, install path, and current status.

**Setup guidance:**

```bash
openclaw skills info research-intel
```

Output includes any missing configuration and the exact steps to make the skill Ready. Follow the prompts to enter API keys or set required values.

---

## Creating Custom Skills

### Frontmatter Format

```yaml
---
name: lowercase-with-hyphens
description: One sentence. What the skill does and when to trigger it.
---
```

The `name` must match the directory name. The `description` is used by the Control UI and CLI for display.

### Required Sections

A well-structured SKILL.md includes:

| Section | Purpose |
|---------|---------|
| Heading + intro | What this skill does |
| Trigger conditions | When the agent should activate this skill |
| Step-by-step process | Numbered workflow to follow |
| Output format | Exact structure for results |
| Escalation rules | When to stop and escalate |

### References Convention

Move long-form content out of SKILL.md into `references/`:

```
skills/<skill-name>/
├── SKILL.md              # Concise instructions (under 200 lines ideal)
└── references/
    ├── examples.md       # Sample outputs
    ├── glossary.md       # Domain terms
    └── templates.md      # Reusable formats
```

Reference them from SKILL.md with relative paths: `See references/examples.md for sample outputs.`

---

## Using Templates from This Repo

The `templates/skills/` directory includes seven pre-built skills:

| Template | Agent | Purpose |
|----------|-------|---------|
| `coding-handoff` | Coder, QA, DevOps | Branch/PR handoffs with ACK/DONE/BLOCKED status |
| `research-intel` | Researcher | Market scans with confidence scoring |
| `leadgen-qualification` | Lead Gen | ICP scoring and lead enrichment |
| `content-repurpose` | Content | Multi-platform content variants |
| `ops-triage` | Ops | Inbox/calendar/task priority routing |
| `telegram-topic-setup` | Orchestrator | Automated Telegram topic creation and agent binding |
| `acpx-session` | Coder | ACPX session management for coding subagent delegation |

These templates predate the native Skills system but use the same YAML frontmatter + markdown format, so they are fully compatible.

**To use a template:**

1. Copy the skill directory into your agent's workspace:
   ```
   cp -r templates/skills/coding-handoff ~/.openclaw/workspace/agents/coder/skills/
   ```

2. Customize the frontmatter `name` and `description` if needed.

3. Add any role-specific content under `references/`.

4. Restart or reload the agent — the skill will appear in the Control UI and CLI.

**Alignment notes:** These templates follow the same path convention (`agents/<agent>/skills/<skill-name>/SKILL.md`) and frontmatter schema that the native system expects. No conversion is needed. See `templates/skills/README.md` for the full list and usage guidance.

---

## Integrating Skills into Agents

Skills are referenced from SOUL.md so the agent knows when to activate them.

In the **Shared Context** or **How I Work** section of SOUL.md, add:

```markdown
## Skills

- **coding-handoff** — Use for all branch handoffs to QA and DevOps
- **research-intel** — Use for any market intelligence request
```

**When agents use skills vs. general instructions:**

| Use a Skill | Use SOUL.md |
|-------------|-------------|
| Repeating multi-step workflow | One-time or ad-hoc behavior |
| Strict output format required | Flexible or conversational output |
| Shared across multiple agents | Unique to one agent |
| Needs its own references/examples | Simple enough to describe inline |
| Has external API dependencies | Pure reasoning tasks |

For guidance on writing effective SOUL.md files, see [agent-design-patterns.md](agent-design-patterns.md).

---

## Security Considerations

**API key management:** Skills that require external keys (e.g., CRM access, analytics APIs) store them in the agent's secure vault — never in SKILL.md or workspace files. The Control UI and CLI handle key entry without exposing values.

**Install source validation:** OpenClaw validates skill installer metadata against strict regex allowlists. URLs are checked against protocol allowlists (HTTPS only). This prevents arbitrary code execution from untrusted sources.

**Sandbox implications:** Skills run within the agent's existing sandbox. A skill cannot grant the agent new capabilities beyond what its configuration allows. Review a skill's required permissions before installing — the `openclaw skills info` command lists all requirements.

---

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| Skill not loading | SKILL.md not at expected path | Verify path is `agents/<agent>/skills/<name>/SKILL.md` and `name` in frontmatter matches directory name |
| Skill shows "Needs Setup" | Missing required API key or config | Open the skill in Control UI or run `openclaw skills info <name>` to see what is missing |
| Agent ignores skill instructions | Skill not referenced in SOUL.md | Add the skill to the agent's SOUL.md under Skills or Shared Context |
| API key invalid after entry | Key revoked or wrong environment | Re-enter the key via Control UI detail dialog |
| Skill install fails from ClawHub | Network or validation error | Check connectivity; verify the skill is from a trusted ClawHub publisher |
| Multiple skills conflict | Overlapping trigger conditions | Clarify in SOUL.md which skill takes priority, or split into separate agents |

For workspace setup and agent configuration, see [INSTRUCTIONS.md](../INSTRUCTIONS.md).
