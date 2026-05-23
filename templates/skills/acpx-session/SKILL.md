---
name: acpx-session
description: Manage ACPX sessions for delegating tasks from an orchestrator agent to coding agents. Use for agent-to-coding-agent work delegation with named sessions, parallel workstreams, and status tracking.
version: "1.0.0"
---

## Install

Copy to `agents/<coder>/skills/acpx-session/SKILL.md`. Requires ACPX CLI installed and at least one coding agent configured.

# ACPX Session Skill

## Session Creation

Create a new session for a coding agent:

```
acpx <agent> sessions new
```

Named sessions for tracking:

```
acpx <agent> sessions new --name <name>
```

## Prompt Submission

Send a task to the agent:

```
acpx <agent> "task description"
```

Submit from a file:

```
acpx <agent> --file <path>
```

Pipe via stdin:

```
echo "task" | acpx <agent>
```

## Parallel Workstreams

Use named sessions to run concurrent work without collisions:

```
acpx <agent> -s frontend "implement login page"
acpx <agent> -s backend "add auth endpoint"
```

Each named session maintains independent context and state.

## Exec Mode

For one-shot, stateless tasks that don't need session persistence:

```
acpx <agent> exec "one-shot task"
```

No session is created or retained after completion.

## Status and Cancel

Check active sessions:

```
acpx <agent> status
```

Cancel a running session:

```
acpx <agent> cancel
```

Cancel a named session:

```
acpx <agent> cancel -s <name>
```

## Integration with sessions_send

Combine ACPX with the handoff protocol:

1. Receive a `HANDOFF` message via `sessions_send`
2. Spawn an ACPX session for the target coding agent
3. Post `ACK` with the session name and ETA
4. On completion, post `DONE` with evidence back via `sessions_send`
5. If blocked, post `BLOCKED` with explicit unblock requirements

This bridges the inter-agent handoff protocol with ACPX session management.

## Operating Rules

When to use which mode:

- **`mode: persistent`** for any task that spans more than one prompt or needs the coding agent to remember context (refactors, multi-file changes, anything with iteration). Cost: a live session per topic.
- **`exec`** for stateless, one-shot work (a single read, a one-prompt summary, a quick syntax fix). No session is retained. Always prefer `exec` when the task fits in one prompt and doesn't need memory of prior turns.

Session naming convention:

- Persistent sessions get a name tied to the task: `<task_id>` or `<scope>-<task_id>` (e.g. `auth-build-142`). This makes ACK/DONE handoffs traceable and lets parallel work coexist without collisions.
- Never reuse a session name across unrelated tasks.

Parallel sessions:

- Use named sessions to keep concurrent workstreams isolated (`acpx codex -s frontend ...` and `acpx codex -s backend ...` run in parallel).
- One named session = one workstream. Don't multiplex unrelated prompts through the same session — the coding agent's context will fragment.

Cancel vs. let it run:

- Cancel cooperatively (`acpx <agent> cancel -s <name>`) when scope changed and the in-flight work is now wrong. Don't kill blindly — cooperative cancel lets the agent flush partial state cleanly.
- Let it run if the work is still relevant but slow. The session lifecycle uses `queued`/`running` states; new prompts queue rather than interrupt.

Failure budget:

- After 3 failed retries on the same prompt, stop. Post `BLOCKED` with what was tried, not another retry. ACPX is not magic — three identical failures usually mean the prompt or the environment is wrong, not the model.

Telegram-specific note:

- For Telegram topics, `/acp spawn <agent> --thread here` is the in-chat way to start a persistent session bound to the current topic. `/acp spawn --bind here` does **not** work on Telegram (it works on Discord/BlueBubbles/iMessage).
- `sessions_spawn(runtime="acp", thread:true)` from a Telegram subagent currently fails ([openclaw/openclaw#41004](https://github.com/openclaw/openclaw/issues/41004)). Use the slash command or a persistent ACP binding in `openclaw.json` instead.

## Supported Agents

| Agent    | Identifier |
|----------|------------|
| Codex    | codex      |
| Claude   | claude     |
| Gemini   | gemini     |
| OpenCode | opencode   |
| Pi       | pi         |
| Copilot  | copilot    |
| Cursor   | cursor     |
| Droid    | droid      |
| Kimi     | kimi       |
| Kiro     | kiro       |
| Qwen     | qwen       |
| Trae     | trae       |
