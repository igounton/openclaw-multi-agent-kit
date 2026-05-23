# INSTRUCTIONS.md — AI Agent Setup Guide

*This file is written for AI agents. If you are an AI agent being asked to set up a multi-agent Telegram supergroup team on OpenClaw, follow these instructions step by step.*

---

## What You Are Building

A team of autonomous AI agents, each with their own Telegram bot, running on OpenClaw. The agents are organized into teams that share topic channels in a Telegram supergroup. They coordinate through shared markdown files and trigger each other using `sessions_send`.

## Prerequisites

Before starting, confirm with the human that they have:

- [ ] OpenClaw installed and running (`openclaw --version`)
- [ ] An LLM API key configured (Anthropic, OpenAI, or other provider in openclaw.json)
- [ ] A Telegram account
- [ ] Access to the OpenClaw config file (`~/.openclaw/openclaw.json`)

---

## Phase 1: Plan the Team

Ask the human what agents they need. Here are the standard roles:

| Role | What It Does | Priority |
|------|-------------|----------|
| **Orchestrator** | Coordinates all agents, delegates tasks | Required |
| **Coder** | Writes, debugs, architects code | High |
| **QA** | Tests and validates builds | High |
| **DevOps** | Deploys and manages infrastructure | Medium |
| **Researcher** | Market research and competitive intel | Medium |
| **Growth** | Analytics, experiments, funnel optimization | Medium |
| **Content** | Social media posts (Twitter/X) | Optional |
| **Community** | Reddit/forum engagement | Optional |
| **Lead Gen** | Prospect research and scoring | Optional |
| **Ops** | Email triage, calendar, data management | Optional |

**Minimum viable team:** Orchestrator + 1 specialist (e.g., Coder).
**Recommended starter team:** Orchestrator + Coder + QA (3 agents).
**Full team:** All 10 agents organized into teams.

For each agent the human wants, you need:
1. A name (e.g., "Connor", "Echo", "Atlas")
2. A Telegram bot (created via @BotFather)
3. A workspace directory
4. A SOUL.md and IDENTITY.md file

---

## Phase 2: Create Telegram Bots

For each agent, the human needs to do this in Telegram (you cannot do this programmatically):

1. Open Telegram, go to [@BotFather](https://t.me/BotFather)
2. Send `/newbot`
3. Set the bot name (e.g., "Connor Bot")
4. Set the bot username (e.g., `connor_myteam_bot`)
5. **Save the bot token** — you will need it for the config
6. Send `/setjoingroups` and select the bot — set to **Enable**
7. Send `/setprivacy` and select the bot — set to **Disable** (so the bot can read group messages)

Repeat for every agent. Collect all bot tokens.

**Tell the human:** "I need you to create [N] Telegram bots via @BotFather. For each one, give me the bot token. Here are the names I suggest: [list]"

---

## Phase 3: Create the Supergroup

The human needs to:

1. Create a new Telegram group
2. Go to group Settings > convert to Supergroup
3. Enable Topics (Settings > Topics > Enable)
4. Create topic channels for each team:

| Topic Name | Purpose | Primary Agent | Secondary Agents |
|-----------|---------|---------------|-----------------|
| General | Orchestrator catch-all | Orchestrator | — |
| Build | Code + QA + Deploy | Coder | QA, DevOps |
| Research | Market intel + analytics | Researcher | Growth |
| Social | Content + community | Content | Community |
| Leads | Prospect research | Lead Gen | — |
| Ops | Email, calendar, data | Ops | — |

5. Add ALL bots to the supergroup as admins
6. Get the group ID (the human can use @raw_data_bot or check OpenClaw logs after adding a bot)
7. Get each topic ID (visible in the Telegram URL when clicking a topic, or from bot update logs)

**Tell the human:** "I need the supergroup ID and the topic ID for each channel you created."

---

## Phase 4: Create Agent Workspaces

For each agent, create the workspace directory and files:

```bash
# Create directories
mkdir -p ~/.openclaw/workspace/agents/{AGENT_NAME}/

# Create shared context (only once)
mkdir -p ~/.openclaw/workspace/shared-context/
```

### Copy and customize templates

For each agent, copy the matching SOUL.md template from `templates/soul/` and the identity template from `templates/identity/agent-identity.md` into their workspace.

**Customization checklist for each SOUL.md:**
- [ ] Replace `[Your ... Name]` with the actual agent name
- [ ] Replace `[Name]` throughout with the agent name
- [ ] Write a tagline that fits the personality
- [ ] Customize the team integration section with actual agent IDs and topic numbers
- [ ] Update `sessions_send` examples with real agent IDs

**Customization checklist for each IDENTITY.md:**
- [ ] Fill in name, role, vibe, model, emoji, channel
- [ ] List actual capabilities
- [ ] Set escalation targets

### Create shared context files

Copy these templates to `~/.openclaw/workspace/shared-context/`:
- `THESIS.md` — Have the human fill in their business thesis
- `SIGNALS.md` — Starts empty, research agents populate it
- `FEEDBACK-LOG.md` — Starts empty, gets filled over time

### Update the supergroup map

Copy `templates/workspace/SUPERGROUP-MAP.md` to `~/.openclaw/workspace/shared-context/` and fill in:
- Actual topic numbers
- Actual agent names and bot accounts
- Team assignments

---

## Phase 5: Configure openclaw.json

This is the critical step. Edit `~/.openclaw/openclaw.json` to add agents, bindings, and channel accounts.

**ALWAYS back up first:**
```bash
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.bak
```

### 5.1 Add agents to the agents list

In the `agents.list` array, add each agent:

```json
{
  "id": "AGENT_ID",
  "workspace": "/home/USER/.openclaw/workspace/agents/AGENT_ID/"
}
```

**Model selection guidance:**

Leave `model` unset on each agent so they inherit the OpenClaw runtime default. The templates in this repo are intentionally model-agnostic — they run on any provider/model your OpenClaw is configured with.

Only pin a specific model on an individual agent if you have a concrete reason:
- A role that consistently underperforms on the default — pin a more capable model just for that role.
- A high-volume, low-stakes role (triage, ack-only handoffs) — pin a smaller/cheaper model to save cost.
- A role that needs a specific provider-only feature.

When you do pin, use the model id format your provider expects, e.g. `"model": "anthropic/<id>"` or `"model": "openai/<id>"`. Treat the per-agent override as the exception, not the default.

### 5.2 Add bindings

In the `bindings` array, map each agent to their Telegram account:

```json
{
  "agentId": "AGENT_ID",
  "match": {
    "channel": "telegram",
    "accountId": "AGENT_ID"
  }
}
```

### 5.3 Add Telegram channel accounts

In `channels.telegram.accounts`, add each agent's bot config:

**For the orchestrator (default account):**
```json
"default": {
  "botToken": "ORCHESTRATOR_BOT_TOKEN",
  "dmPolicy": "pairing",
  "groups": {
    "GROUP_ID": {
      "requireMention": false,
      "groupPolicy": "open",
      "enabled": true,
      "topics": {
        "1": { "requireMention": false, "groupPolicy": "open", "enabled": true },
        "TOPIC_BUILD": { "enabled": false },
        "TOPIC_RESEARCH": { "enabled": false },
        "TOPIC_SOCIAL": { "enabled": false },
        "TOPIC_LEADS": { "enabled": false },
        "TOPIC_OPS": { "enabled": false }
      }
    }
  },
  "groupPolicy": "allowlist",
  "streaming": "off"
}
```

**Key:** The orchestrator's topics config should have `enabled: false` for topics owned by other agents. This prevents the orchestrator from responding in agent-owned topics.

**For an agent on a single-agent topic (primary, no teammates):**
```json
"AGENT_ID": {
  "botToken": "AGENT_BOT_TOKEN",
  "dmPolicy": "pairing",
  "groups": {
    "GROUP_ID": {
      "requireMention": true,
      "groupPolicy": "open",
      "enabled": true,
      "topics": {
        "TOPIC_NUMBER": {
          "requireMention": false,
          "groupPolicy": "open",
          "enabled": true
        }
      }
    }
  },
  "groupPolicy": "allowlist",
  "streaming": "off"
}
```

**Key:** `requireMention: true` at the group level means the bot only responds when @mentioned in most topics. But `requireMention: false` on their specific topic means they respond to ALL messages in their own topic. **This only works when the agent is the ONLY bot in that topic.**

**For a secondary agent on a team topic:**
```json
"AGENT_ID": {
  "botToken": "AGENT_BOT_TOKEN",
  "dmPolicy": "pairing",
  "groups": {
    "GROUP_ID": {
      "requireMention": true,
      "groupPolicy": "open",
      "enabled": true,
      "topics": {
        "TOPIC_NUMBER": {
          "requireMention": true,
          "groupPolicy": "open",
          "enabled": true
        }
      }
    }
  },
  "groupPolicy": "allowlist",
  "streaming": "off"
}
```

**Key:** Secondary agents have `requireMention: true` even on their team topic. They only respond when @mentioned by the human or triggered via `sessions_send` by a teammate.

**Important:** In topics with multiple bots, ALL bots must have `requireMention: true`. If one bot has `requireMention: false`, it will respond to every message — including when you're trying to @mention another agent. This causes multiple bots to respond to the same message.

**For all agents in a multi-agent topic:**
```json
"TOPIC_NUMBER": {
  "requireMention": true,
  "groupPolicy": "open",
  "enabled": true
}
```

**Every agent** sharing that topic must have this exact config. No exceptions.

### 5.4 Add ACP config (optional)

If you want agents to spawn coding subagents via ACPX:

```json
"acp": {
  "enabled": true,
  "backend": "acpx",
  "defaultAgent": "claude",
  "allowedAgents": ["claude", "codex", "opencode", "gemini", "pi"]
}
```

This lets the coder agent use `sessions_spawn(runtime="acp")` to delegate to Claude Code, Codex, or other ACP-compatible agents.

### 5.5 Add thread bindings (recommended)

Enable persistent sessions per topic. Thread bindings live **under the channel** (`channels.telegram.threadBindings`), with an optional per-account override. They do NOT live at top-level `session` — that's a different block.

Channel-level default (applies to all Telegram accounts unless overridden):

```json
"channels": {
  "telegram": {
    "threadBindings": {
      "spawnSessions": true,
      "idleHours": 48
    }
  }
}
```

Per-account override (e.g., only the coder account should spawn ACP sessions on new threads):

```json
"channels": {
  "telegram": {
    "accounts": {
      "coder": {
        "threadBindings": {
          "spawnSessions": true
        }
      }
    }
  }
}
```

The only documented keys for Telegram `threadBindings` are `spawnSessions` (boolean, default `true`) and `idleHours`. Keys like `enabled`, `maxAgeHours`, or `maxAgeDays` are not part of the current Telegram schema — leave them out.

### 5.6 Configure groupAllowFrom

For each account, add the human's Telegram user ID to restrict who can interact:

```json
"groupAllowFrom": ["HUMAN_TELEGRAM_USER_ID"]
```

---

## Phase 6: Set Up Bot-to-Bot Communication

Telegram bots **cannot see each other's messages**. This is a Telegram platform limitation.

Agents communicate with each other using OpenClaw's `sessions_send`:

```
sessions_send(agentId="TARGET_AGENT", message="HANDOFF ...")
```

Use the strict handoff format from [`docs/inter-agent-handoff-standard.md`](docs/inter-agent-handoff-standard.md).

**Minimum required fields in every handoff:**
- `from`
- `to`
- `task_id`
- `priority`
- `summary`
- `context`
- `deliver_to`
- `deadline`
- `done_when` (acceptance criteria)

The receiving agent gets this as a new session input. They then post their response in the shared topic using the `message` tool:

```
message(action="send", channel="telegram", target="GROUP_ID/TOPIC_NUMBER", message="ACK <task_id> ...")
```

### Workflow example: Build Team

```
1. Human posts task in Build topic
2. Coder (primary) picks it up, builds the feature
3. Coder posts summary in topic (human can see)
4. Coder triggers QA:
   sessions_send(agentId="qa", message="QA request for Build topic:\n- What: [description]\n- Branch: [branch]\nPost verdict in topic.")
5. QA tests, posts verdict in topic (human can see)
6. If PASS: QA triggers DevOps:
   sessions_send(agentId="devops", message="Deploy request:\n- QA: PASSED\n- Branch: [branch]\nPost status in topic.")
7. If FAIL: Coder fixes, re-triggers QA
```

### Workflow example: Research Team

```
1. Human or cron triggers Researcher with a brief
2. Researcher investigates, writes findings to SIGNALS.md
3. Researcher posts summary in Research topic
4. Researcher triggers Growth for analysis:
   sessions_send(agentId="growth", message="Analysis request:\n- Data: [findings]\n- Need: [what analysis]\nPost analysis in topic.")
5. Growth analyzes and posts in Research topic
```

### Workflow example: Social Team

```
1. Content agent drafts posts based on SIGNALS.md
2. Content posts drafts in Social topic for human review
3. When content resonates on Twitter, Content triggers Community:
   sessions_send(agentId="community", message="Cross-post opportunity:\n- Content: [what worked]\n- Angle: [how to adapt]\nPost plan in topic.")
4. Community adapts for Reddit, posts plan in Social topic
5. Community intel flows back to Content via SIGNALS.md
```

**Important:** Each agent's SOUL.md must document:
- Which agents they trigger and when
- The exact `sessions_send` format they use
- Which topic they post results to (with group ID and topic number)
- What triggers them (mention or sessions_send)

---

## Phase 7: Restart and Test

```bash
openclaw restart
```

### Test checklist:

- [ ] Send a message in the General topic — orchestrator responds
- [ ] Send a message in the Build topic — coder responds (not orchestrator)
- [ ] @mention a secondary agent in their team topic — they respond
- [ ] Test `sessions_send` between two agents — receiving agent gets the message
- [ ] Verify agents post to topics using the `message` tool
- [ ] Verify the orchestrator does NOT respond in agent-owned topics
- [ ] Send a DM to each bot — they respond (if dmPolicy is set)

### Common issues:

| Problem | Cause | Fix |
|---------|-------|-----|
| Bot not responding | Bot not admin in group | Add bot as admin |
| Bot responds in wrong topic | Topic config wrong | Check `requireMention` and `enabled` per topic |
| Bot responds to everything | `requireMention: false` at group level | Set to `true`, override per topic |
| sessions_send not working | Agent ID mismatch | Check `agents.list` IDs match `sessions_send` targets |
| Orchestrator responds in agent topics | Topic not disabled for orchestrator | Set `enabled: false` for agent topics in default account |
| Bot ignores messages | `groupPolicy: allowlist` but no `groupAllowFrom` | Add human's Telegram user ID |

---

## Phase 8: Optional Enhancements

### Skills setup

Install skills for your agents:

```bash
# List available skills
openclaw skills list

# Get details on a skill
openclaw skills info coding-handoff
```

Or copy skill templates from `templates/skills/` into agent workspaces:

```bash
cp templates/skills/coding-handoff/SKILL.md ~/.openclaw/workspace/agents/coder/skills/coding-handoff/SKILL.md
```

See [docs/skills-system.md](docs/skills-system.md) for the native Skills system.

### ACPX setup

If you skipped ACP in Phase 5.4 and want to add coding subagent delegation later:

```bash
npm install -g acpx@latest
acpx --version
```

Then add the `acp` config block from Phase 5.4 to your `openclaw.json`. See [docs/acpx-telegram.md](docs/acpx-telegram.md) for the full guide.

### Cron jobs

Schedule recurring tasks:

```json
"cron": {
  "jobs": [
    {
      "agentId": "ops",
      "schedule": "45 * * * *",
      "message": "Check Gmail for new important emails. Summarize and flag urgent items."
    },
    {
      "agentId": "researcher",
      "schedule": "0 8 * * *",
      "message": "Run daily market research. Write findings to SIGNALS.md and post summary in topic."
    }
  ]
}
```

### Memory search

Enable semantic memory search across agent memory files:

```json
"memorySearch": {
  "enabled": true,
  "provider": "gemini",
  "model": "gemini-embedding-001"
}
```

### Internal hooks

Enable boot-time file loading and memory indexing:

```json
"hooks": {
  "internal": {
    "enabled": true,
    "entries": {
      "boot-md": { "enabled": true },
      "bootstrap-extra-files": { "enabled": true },
      "session-memory": { "enabled": true }
    }
  }
}
```

### ACP (Agent Coding Platform)

If you want agents to spawn Claude Code for coding tasks:

```json
"acp": {
  "enabled": true,
  "backend": "acpx",
  "defaultAgent": "claude",
  "allowedAgents": ["claude"]
}
```

The coder agent can then use:
```
sessions_spawn(runtime="acp", agentId="claude", cwd="/path/to/repo")
```

---

## Summary Checklist

- [ ] Team planned (which agents, which teams)
- [ ] Telegram bots created (one per agent, tokens saved)
- [ ] Supergroup created with topics enabled
- [ ] Topics created for each team
- [ ] All bots added to supergroup as admin
- [ ] Group ID and topic IDs collected
- [ ] Agent workspaces created with SOUL.md + IDENTITY.md
- [ ] Shared context files created (THESIS.md, SIGNALS.md, FEEDBACK-LOG.md)
- [ ] SUPERGROUP-MAP.md filled in
- [ ] openclaw.json backed up
- [ ] openclaw.json updated (agents, bindings, channel accounts)
- [ ] OpenClaw restarted
- [ ] All agents tested in correct topics
- [ ] Bot-to-bot communication tested
- [ ] Cron jobs configured (if needed)
- [ ] Human briefed on what each agent does and where to find them
- [ ] ACP config added (if using coding subagents)
- [ ] Thread bindings configured
- [ ] Skills installed or copied to agent workspaces
- [ ] ACPX installed and tested (if using coding subagents)

---

*This guide assumes OpenClaw 2026.x. If the config format has changed, check the OpenClaw docs.*
