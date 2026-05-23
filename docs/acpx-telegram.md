# ACPX + Telegram Integration Guide

How to connect **ACPX** (the headless CLI for Agent Client Protocol) to your OpenClaw Telegram agent team, giving your agents direct access to coding agents like Claude Code, Codex, OpenCode, Gemini CLI, and more.

> **Alpha status.** ACPX is in alpha. Config keys, CLI flags, and behavior may change between releases. Pin your version in production and test upgrades before deploying.

For basic ACP topic binding in DM conversations, see [telegram-dm-topics.md](telegram-dm-topics.md). This document is the full guide covering persistent bindings, exec mode, flows, and multi-agent patterns.

---

## What ACPX Does

ACPX (`npm: acpx`) is the headless CLI client for stateful Agent Client Protocol (ACP) sessions. Instead of spawning a terminal and scraping PTY output, OpenClaw agents delegate coding work to coding agents over a structured protocol.

Key capabilities:

- **Persistent sessions** with conversation history — coding agents remember context across messages
- **Named sessions** for parallel workstreams (e.g. `auth-refactor`, `bugfix-42`)
- **Prompt queueing and cooperative cancel** — don't lose work if a new prompt arrives mid-task
- **One-shot exec mode** — fire-and-forget commands without session overhead
- **Flows** — multi-step automated workflows from a single trigger
- **12 built-in agent backends**: claude, codex, opencode, gemini, pi, copilot, cursor, droid, kimi, kiro, qwen, trae

---

## Prerequisites

| Requirement | Details |
|---|---|
| **ACPX** | `npm install -g acpx@latest` |
| **Agent backend(s)** | At least one installed: Claude Code, Codex, OpenCode, Gemini CLI, etc. |
| **OpenClaw** | v2026.3.28 or later (adds `/acp spawn`, persistent ACP bindings); v2026.3.31 is the latest known release |
| **Telegram supergroup** | Configured per [supergroup-setup.md](supergroup-setup.md) or [telegram-dm-topics.md](telegram-dm-topics.md) |
| **Node.js** | 18+ (for ACPX) |

Verify ACPX is installed:

```bash
acpx --version
```

List available agent backends:

```bash
acpx agents
```

---

## How It Works

The ACP protocol routes messages through three layers:

```
Telegram message → OpenClaw gateway → ACPX session → Coding agent
```

1. A message arrives in a Telegram topic or DM
2. OpenClaw matches the conversation against an ACP binding
3. The gateway forwards the message to ACPX as a prompt
4. ACPX delivers it to the coding agent (Claude Code, Codex, etc.)
5. The coding agent's response flows back through ACPX → gateway → Telegram

**Session lifecycle:**

| State | Meaning |
|---|---|
| `idle` | Session exists, no active prompt |
| `running` | Coding agent is processing a prompt |
| `queued` | Prompt is waiting behind an active one |
| `cancelled` | Cooperative cancel was issued, agent is winding down |
| `closed` | Session terminated |

Sessions persist until they time out (configurable via `idleHours`) or the gateway restarts.

---

## Setting Up ACPX for Telegram

### Step 1 — Install ACPX

```bash
npm install -g acpx@latest
```

Optionally configure ACPX defaults:

```bash
acpx config set defaultAgent codex
acpx config set defaultMode persistent
```

ACPX reads from `~/.acpx/config.json` (global) and `.acpxrc.json` (project-level). Project config overrides global.

### Step 2 — Add an ACP agent to `openclaw.json`

Add a new agent entry with `runtime.type: "acp"`:

```json
{
  "agents": {
    "list": [
      {
        "id": "coder-acp",
        "workspace": "/home/YOUR_USER/.openclaw/workspace/agents/coder/",
        "runtime": {
          "type": "acp",
          "acp": {
            "backend": "acpx",
            "agent": "codex",
            "mode": "persistent",
            "cwd": "/home/YOUR_USER/projects/your-app"
          }
        }
      }
    ]
  }
}
```

| Field | Description |
|---|---|
| `runtime.type` | Must be `"acp"` for ACPX-backed agents |
| `acp.backend` | `"acpx"` to use the ACPX CLI |
| `acp.agent` | Agent backend name: `codex`, `claude`, `opencode`, `gemini`, etc. |
| `acp.mode` | `"persistent"` for session reuse, `"exec"` for one-shot |
| `acp.cwd` | Working directory for the coding agent |

### Step 3 — Add a persistent ACP binding

In the `bindings` array, bind the agent to a Telegram conversation:

```json
{
  "bindings": [
    {
      "type": "acp",
      "agentId": "coder-acp",
      "match": {
        "channel": "telegram",
        "accountId": "default",
        "peer": {
          "kind": "group",
          "id": "YOUR_GROUP_ID:topic:TOPIC_BUILD"
        }
      },
      "acp": {
        "label": "build-coder",
        "mode": "persistent",
        "cwd": "/home/YOUR_USER/projects/your-app"
      }
    }
  ]
}
```

The `match` object determines which conversations route to this ACP agent. Use the same `channel`, `accountId`, and `peer` pattern as regular agent bindings.

### Step 4 — Enable the topic in channel config

```json
{
  "channels": {
    "telegram": {
      "accounts": {
        "default": {
          "groups": {
            "YOUR_GROUP_ID": {
              "topics": {
                "TOPIC_BUILD": {
                  "requireMention": false,
                  "groupPolicy": "open",
                  "enabled": true
                }
              }
            }
          }
        }
      }
    }
  }
}
```

### Step 5 — Enable thread bindings

Thread bindings allow OpenClaw to automatically create ACP sessions when conversations start in a bound topic. They live under `channels.telegram.threadBindings` (channel-wide) and/or `channels.telegram.accounts.<accountId>.threadBindings` (per-account override). They are **not** under top-level `session.threadBindings` — that path is not part of the current schema.

```json
{
  "channels": {
    "telegram": {
      "threadBindings": {
        "spawnSessions": true,
        "idleHours": 48
      },
      "accounts": {
        "default": {
          "threadBindings": {
            "spawnSessions": true
          }
        }
      }
    }
  }
}
```

`spawnSessions: true` is the key setting (default `true`) — it tells the gateway to create an ACPX session when a new thread starts in a topic with an ACP binding. The only other documented Telegram `threadBindings` key is `idleHours`. Older keys like `enabled`, `spawnAcpSessions`, `maxAgeHours`, or `maxAgeDays` are not part of the Telegram schema.

### Step 6 — Enable ACP globally

```json
{
  "acp": {
    "enabled": true,
    "backend": "acpx",
    "allowedAgents": ["codex", "claude", "opencode"]
  }
}
```

Only agents listed in `allowedAgents` can be spawned. Add every backend you intend to use.

### Step 7 — Restart the gateway

```bash
openclaw gateway restart
```

After restart, messages in the bound topic route directly to the ACPX session. Verify by sending a test message to the topic.

---

## The `/acp spawn --thread here` Pattern

New in OpenClaw v2026.3.28.

The `/acp spawn` slash command lets you turn the current Telegram forum topic into a coding workspace on the fly:

```text
/acp spawn codex --thread here
```

This creates a persistent ACPX session bound to the **current topic** — no config changes needed. Every subsequent message in that topic goes to the coding agent until the session expires. Use `--thread auto` to let the gateway pick the binding mode.

> **Telegram-specific flag note.** The general `/acp spawn` command supports both `--bind here|off` and `--thread auto|here|off`, but **Telegram channel sessions cannot use `--bind here`** — only thread-bound modes work when `channels.telegram.threadBindings.spawnSessions` is enabled (the default). For Discord, BlueBubbles, and iMessage, `--bind here` is supported.

**How it differs from persistent bindings:**

| Aspect | Persistent binding | `--thread here` |
|---|---|---|
| Setup | Config in `openclaw.json` | Slash command in chat |
| Scope | Pre-defined topics/peers | Current topic only |
| Persistence | Survives gateway restart | Session-scoped, expires with idle timeout |
| Use case | Production workflows | Ad-hoc coding tasks, debugging |

---

## Multi-Agent ACPX Patterns

An orchestrator or coder agent can delegate work to an ACPX-backed coding agent programmatically via `sessions_spawn`:

```text
sessions_spawn(
  runtime="acp",
  agent="codex",
  prompt="Refactor the auth module to use JWT refresh tokens"
)
```

**Important restriction:** `sessions_spawn(runtime="acp")` can only be called from a `subagent:*` session — not directly from a Telegram channel session. This is a platform-level restriction.

To use programmatic ACP spawning from Telegram:

1. The human or orchestrator sends a message in a topic
2. The topic's primary agent spawns a subagent via `sessions_send`
3. The subagent calls `sessions_spawn(runtime="acp")` to spawn the ACPX session
4. The coding agent does the work
5. The subagent posts the result back in the topic

For chat-driven ACP sessions (no subagent intermediary), use the `/acp spawn` slash command or configure persistent bindings.

---

## ACPX Exec Mode

For one-shot tasks that don't need persistent context:

```bash
acpx codex exec "summarize this repo"
```

In `openclaw.json`, configure an agent with `mode: "exec"`:

```json
{
  "id": "coder-exec",
  "runtime": {
    "type": "acp",
    "acp": {
      "backend": "acpx",
      "agent": "codex",
      "mode": "exec",
      "cwd": "/home/YOUR_USER/projects/your-app"
    }
  }
}
```

Exec mode is useful for:

- Quick codebase queries from Telegram
- Running linting, tests, or build checks
- Generating summaries or reports
- Any task where you don't need the coding agent to remember prior context

---

## ACPX Flows

Flows chain multiple prompts into a single automated workflow:

```bash
acpx flow run /path/to/flow.yaml
```

Example flow file:

```yaml
name: code-review-flow
steps:
  - agent: codex
    prompt: "Run the full test suite and report failures"
  - agent: codex
    prompt: "For each failing test, propose a fix"
  - agent: codex
    prompt: "Apply the fixes and re-run tests"
```

Trigger a flow from Telegram by binding it to an agent or invoking via a subagent. Flows are ideal for complex multi-step coding tasks — code review, migration scripts, batch refactoring — that require sequential prompts with shared context.

---

## Integrating with the Handoff Protocol

ACPX sessions integrate naturally with the inter-agent handoff standard (see [inter-agent-handoff-standard.md](inter-agent-handoff-standard.md)).

**Typical flow:**

1. The orchestrator sends a `HANDOFF` to the coder agent via `sessions_send`
2. The coder agent posts `ACK <task_id>` in the Build topic
3. The coder spawns an ACPX session (via subagent) to do the actual coding work
4. The coding agent (Codex, Claude Code, etc.) completes the implementation
5. The coder agent posts `DONE <task_id>` with results in the topic

Example handoff that triggers ACPX work:

```text
HANDOFF
from: orchestrator
to: coder
task_id: auth-jwt-001
priority: P1
summary: Implement JWT refresh token rotation
context: See docs/auth-spec.md for requirements. Branch: feature/jwt-refresh
deliver_to: telegram:YOUR_GROUP_ID:TOPIC_BUILD
deadline: asap
done_when:
- Refresh token endpoint returns 200
- Token rotation tests pass
- No regression in existing auth tests
```

The coder agent receives this, ACKs, spawns an ACPX session with the coding agent, and posts DONE when the coding agent finishes.

---

## Complete Config Example

Full `openclaw.json` snippet showing all ACPX sections together:

```json
{
  "agents": {
    "list": [
      {
        "id": "orchestrator",
        "workspace": "/home/YOUR_USER/.openclaw/workspace/",
        "model": "YOUR_PROVIDER/YOUR_MODEL_ID",
        "subagents": { "allowAgents": ["*"] }
      },
      {
        "id": "coder-acp",
        "workspace": "/home/YOUR_USER/.openclaw/workspace/agents/coder/",
        "model": "YOUR_PROVIDER/YOUR_MODEL_ID",
        "runtime": {
          "type": "acp",
          "acp": {
            "backend": "acpx",
            "agent": "codex",
            "mode": "persistent",
            "cwd": "/home/YOUR_USER/projects/your-app"
          }
        }
      },
      {
        "id": "coder-exec",
        "workspace": "/home/YOUR_USER/.openclaw/workspace/agents/coder/",
        "runtime": {
          "type": "acp",
          "acp": {
            "backend": "acpx",
            "agent": "codex",
            "mode": "exec",
            "cwd": "/home/YOUR_USER/projects/your-app"
          }
        }
      }
    ]
  },

  "bindings": [
    {
      "type": "acp",
      "agentId": "coder-acp",
      "match": {
        "channel": "telegram",
        "accountId": "default",
        "peer": {
          "kind": "group",
          "id": "YOUR_GROUP_ID:topic:TOPIC_BUILD"
        }
      },
      "acp": {
        "label": "build-coder",
        "mode": "persistent",
        "cwd": "/home/YOUR_USER/projects/your-app"
      }
    }
  ],

  "acp": {
    "enabled": true,
    "backend": "acpx",
    "allowedAgents": ["codex", "claude", "opencode"]
  },

  "channels": {
    "telegram": {
      "enabled": true,
      "threadBindings": {
        "spawnSessions": true,
        "idleHours": 48
      },
      "accounts": {
        "default": {
          "botToken": "YOUR_BOT_TOKEN",
          "dmPolicy": "pairing",
          "groups": {
            "YOUR_GROUP_ID": {
              "requireMention": false,
              "groupPolicy": "open",
              "enabled": true,
              "topics": {
                "1": { "requireMention": false, "enabled": true },
                "TOPIC_BUILD": {
                  "requireMention": false,
                  "groupPolicy": "open",
                  "enabled": true
                }
              }
            }
          },
          "threadBindings": {
            "spawnSessions": true
          }
        }
      }
    }
  }
}
```

---

## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Messages in topic get no response | Binding `match` is wrong | Verify `peer.id` format: `GROUP_ID:topic:TOPIC_ID` |
| `sessions_spawn(runtime="acp", thread:true)` from Telegram returns "Thread bindings do not support ACP thread spawn for telegram" | Known issue ([openclaw/openclaw#41004](https://github.com/openclaw/openclaw/issues/41004)) — direct `/acp spawn` works in-topic but the `thread:true` variant of `sessions_spawn` is not yet supported for Telegram | Use `/acp spawn <agent> --thread here` from the topic, or configure a persistent ACP binding in `openclaw.json` |
| `sessions_send` between agents breaks Telegram routing | Known issue ([openclaw/openclaw#31671](https://github.com/openclaw/openclaw/issues/31671)) — `sessions_send` can corrupt the target session's channel field from `telegram` to `webchat`, so the target agent stops receiving Telegram messages | Until fixed upstream, have receivers post lifecycle messages (ACK/DONE) explicitly via the `message` tool targeting the topic, rather than relying on the target session's channel state |
| ACPX session not created on new thread | `spawnSessions` not set | Set `channels.telegram.threadBindings.spawnSessions: true` (or the per-account override) |
| Coding agent times out | Large codebase or complex task | Increase `idleHours`, or break task into smaller prompts |
| `acpx: command not found` | ACPX not installed globally | Run `npm install -g acpx@latest` |
| `agent not found` error | Backend not installed | Install the coding agent CLI (e.g. Claude Code, Codex CLI) |
| Binding creates duplicate responses | Topic has both regular agent + ACP binding | Use only one binding per topic, or set `requireMention: true` on one |
| `/acp spawn` not recognized | OpenClaw version too old | Upgrade to v2026.3.28+ |
| Session lost after gateway restart | Persistent sessions are in-memory by default | This is expected — use persistent bindings to auto-recreate on restart |
| Wrong working directory | `acp.cwd` not set or incorrect | Set `cwd` in both the agent definition and the binding |
