---
summary: "Forum topics inside a Telegram direct chat + ACP binding"
read_when:
  - "Setting up DM topics instead of (or alongside) a supergroup"
  - "Binding ACP sessions inside a direct chat"
---

# Telegram DM Topics Guide

How to use **Telegram forum topics inside a direct chat** with OpenClaw.

This is different from the usual supergroup/forum setup.

---

## What this is

Telegram now supports forum-style topics in some direct chats. OpenClaw can work with them, but there are a few gotchas:

- each topic behaves like a separate conversation surface
- topic messages need an initial message after creation or Telegram may not visibly show the topic
- access/pairing checks may trigger when a new topic appears for the first time

---

## Creating a topic

### Basic creation

Use the `topic-create` action:

```text
action=topic-create
channel=telegram
target=telegram:<chatId>
name=Coding
```

This returns a `topicId`. Always send a first message into the topic immediately after — otherwise Telegram may not show it in the sidebar.

```text
action=send
channel=telegram
target=telegram:<chatId>
threadId=<topicId>
message=Topic ready.
```

### Adding an icon

Use the `icon_custom_emoji_id` parameter with a valid Telegram forum topic sticker ID.

**Important:** Only specific emoji IDs from Telegram's official forum topic sticker set are valid. Using arbitrary emoji IDs will return `DOCUMENT_INVALID`.

Fetch the full list of valid IDs:

```bash
curl "https://api.telegram.org/bot<TOKEN>/getForumTopicIconStickers"
```

#### Valid icon emoji reference

| Emoji | ID |
|---|---|
| 📰 | 5434144690511290129 |
| 💡 | 5312536423851630001 |
| ⚡️ | 5312016608254762256 |
| 🎙 | 5377544228505134960 |
| 🔝 | 5418085807791545980 |
| 🗣 | 5370870893004203704 |
| 🆒 | 5420216386448270341 |
| ❗️ | 5379748062124056162 |
| 📝 | 5373251851074415873 |
| 📆 | 5433614043006903194 |
| 📁 | 5357315181649076022 |
| 🔎 | 5309965701241379366 |
| 📣 | 5309984423003823246 |
| 🔥 | 5312241539987020022 |
| ❤️ | 5312138559556164615 |
| ❓ | 5377316857231450742 |
| 📈 | 5350305691942788490 |
| 📉 | 5350713563512052787 |
| 💎 | 5309958691854754293 |
| 💰 | 5350452584119279096 |
| 💸 | 5309929258443874898 |
| 🪙 | 5377690785674175481 |
| 💱 | 5310107765874632305 |
| ⁉️ | 5377438129928020693 |
| 🎮 | 5309950797704865693 |
| 💻 | 5350554349074391003 |
| 📱 | 5409357944619802453 |
| 🚗 | 5312322066328853156 |
| 🏠 | 5312486108309757006 |
| 💘 | 5310029292527164639 |
| 🎉 | 5310228579009699834 |
| ‼️ | 5377498341074542641 |
| 🏆 | 5312315739842026755 |
| 🏁 | 5408906741125490282 |
| 🎬 | 5368653135101310687 |
| 🎵 | 5310045076531978942 |
| 🔞 | 5420331611830886484 |
| 📚 | 5350481781306958339 |
| 👑 | 5357107601584693888 |
| ⚽️ | 5375159220280762629 |

#### Example with icon

```text
action=topic-create
channel=telegram
target=telegram:<chatId>
name=Coding
icon_custom_emoji_id=5350554349074391003
```

**Do NOT include emoji in the topic name** — the icon is set separately via `icon_custom_emoji_id`. Putting an emoji in the name results in a plain text emoji with no custom icon.

---

## Recommended creation flow

1. Create the topic with name + icon
2. Capture the returned `topicId`
3. Send an initial message using `threadId=<topicId>`
4. Record the `topicId` for future use

---

## Enabling ACP (Claude Code / Codex / opencode) in a topic

You can bind a Telegram topic to a persistent ACP coding agent so every message in that topic goes directly to a coding harness.

### Step 1 — Add an ACP agent to `openclaw.json`

```json
{
  "agents": {
    "list": [
      {
        "id": "opencode",
        "workspace": "~/.openclaw/workspace/agents/opencode/",
        "runtime": {
          "type": "acp",
          "acp": {
            "agent": "opencode",
            "backend": "acpx",
            "mode": "persistent",
            "cwd": "/your/projects"
          }
        }
      }
    ]
  }
}
```

Supported agents: `claude` (Claude Code), `codex` (Codex CLI), `opencode` (OpenCode), `gemini` (Gemini CLI), `pi` (Pi Coding Agent), `copilot` (GitHub Copilot CLI), `cursor` (Cursor CLI), `droid` (Factory Droid), `kimi` (Kimi CLI), `kiro` (Kiro CLI), `qwen` (Qwen Code), `trae` (Trae CLI).

### Step 2 — Add a persistent ACP binding

```json
{
  "bindings": [
    {
      "type": "acp",
      "agentId": "opencode",
      "match": {
        "channel": "telegram",
        "accountId": "default",
        "peer": { "kind": "group", "id": "<chatId>:topic:<topicId>" }
      },
      "acp": {
        "label": "coding",
        "mode": "persistent",
        "cwd": "/your/projects"
      }
    }
  ]
}
```

Replace `<chatId>` with your chat ID (e.g. `5594479851`) and `<topicId>` with the topic ID returned from `topic-create`.

### Step 3 — Enable the topic in channel config

```json
{
  "channels": {
    "telegram": {
      "accounts": {
        "default": {
          "groups": {
            "<chatId>": {
              "topics": {
                "<topicId>": {
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

### Step 4 — Enable thread bindings

Thread bindings live under `channels.telegram.threadBindings` (and optionally per-account). They are **not** at top-level `session`. The only documented Telegram keys are `spawnSessions` (boolean, default `true`) and `idleHours`.

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

### Step 5 — Ensure ACP is enabled globally

```json
{
  "acp": {
    "enabled": true,
    "backend": "acpx",
    "allowedAgents": ["claude", "codex", "opencode"]
  }
}
```

### Step 6 — Restart the gateway

```bash
openclaw gateway restart
```

After restart, messages sent to that topic route directly to the ACP agent.

### Important: ACP spawn from channel sessions

`sessions_spawn(runtime="acp")` can **only** be called from a `subagent:*` session — not directly from a Telegram channel session. This is a platform restriction.

For programmatic ACP spawning, route through a subagent intermediary:
1. Spawn a subagent
2. Have the subagent call `sessions_spawn(runtime="acp")`

For chat-driven ACP sessions, use the `/acp spawn` slash command or configure persistent bindings as above.

> **Known issue ([openclaw/openclaw#41004](https://github.com/openclaw/openclaw/issues/41004)).** `sessions_spawn(runtime="acp", thread:true)` currently returns *"Thread bindings do not support ACP thread spawn for telegram"* even from a subagent. Until this is fixed upstream, the reliable paths for Telegram are (a) `/acp spawn <agent> --thread here` in-topic, or (b) a persistent ACP binding declared in `openclaw.json` (Steps 2–4 above).

### ACPX Session Management

Once an ACP topic is set up, you can manage coding sessions:

- Named sessions: `acpx codex sessions new --name api` creates a parallel workstream
- Status: `acpx codex status` shows current session state
- Cancel: `acpx codex cancel` cooperatively cancels in-flight prompts
- One-shot: `acpx codex exec "summarize this file"` for stateless tasks

See [acpx-telegram.md](acpx-telegram.md) for the complete guide.

### `/acp spawn --thread here` (v2026.3.28+)

OpenClaw v2026.3.28 added in-topic ACP binds. For Telegram, use:

```
/acp spawn codex --thread here
```

This binds the **current forum topic** to a new ACP session — every subsequent message in that topic goes to the coding agent until the session expires. `--thread auto` is also supported (gateway picks the binding mode).

> **Telegram-specific flag note.** `/acp spawn` documents both `--bind here|off` and `--thread auto|here|off`, but Telegram channel sessions cannot use `--bind here` — only thread-bound modes work when `channels.telegram.threadBindings.spawnSessions` is enabled. Use `--bind here` only from Discord, BlueBubbles, or iMessage. (See the `--bind` flag note in [acpx-telegram.md](acpx-telegram.md#the-acp-spawn---thread-here-pattern).)

Difference from persistent bindings:
- Persistent bindings (Step 2 above) are configured in `openclaw.json` and survive restarts.
- `--thread here` is a runtime command that binds the current topic for the session lifetime (subject to `idleHours`).
- Use persistent bindings for permanent coding topics; use `--thread here` for ad-hoc coding tasks.

---

## How to use DM topics

Once created, treat each topic like a separate thread.

Typical uses:
- `Coding` — ACP coding workspace
- `Research` — market intel, competitor analysis
- `Leads` — lead gen agent output
- `Ops` — system management, deployments
- `Health` — fitness tracking, goals

Use the returned topic/thread id when sending follow-up messages.

---

## Access / pairing gotcha

A newly created topic in a DM may trigger OpenClaw access control or pairing checks.

Why this happens:
- OpenClaw can treat a new DM topic as a new conversation surface
- if access policy is not already configured for that surface, pairing/auth may fire

Typical symptom:
- a pairing or "access not configured" message appears right after using the new topic

What to do:
- approve/pair the new surface if required by your Telegram access policy
- then retry the first message in the topic

---

## Internal owner vs visible sender

If you later route DM topics with native topic `agentId` config:

- `agentId` selects the **internal OpenClaw agent**
- the Telegram account/bot still controls the **visible sender**

So topic routing changes the brain, not the face.

---

## Best practices

- always send an initial message right after topic creation
- use `icon_custom_emoji_id` from the valid sticker set — arbitrary emoji IDs will fail
- do NOT put emoji in the topic name if using a custom icon — they're separate
- record topic IDs if you plan to automate follow-up messages
- expect new-topic pairing behavior in stricter access setups
- for ACP-bound topics, configure persistent bindings in `openclaw.json` rather than using `/acp spawn` each time
- for the full ACPX guide including exec mode, flows, and multi-agent patterns, see [acpx-telegram.md](acpx-telegram.md)

---

## When this is useful

DM topics are great when you want:
- one person chatting privately with one bot/account
- multiple organized lanes inside that same chat (coding, research, ops, etc.)
- ACP coding agents bound to specific topics
- clean separation of context without switching chats

DM topics are not ideal when you want:
- multiple visible specialist bot identities
- public team presence with different bot personas

For that, use the multi-bot supergroup model instead.
