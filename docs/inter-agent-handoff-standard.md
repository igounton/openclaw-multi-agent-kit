---
summary: "HANDOFF/ACK/DONE/BLOCKED contract for all sessions_send cross-agent requests"
read_when:
  - "Writing any cross-agent flow or trigger"
  - "Handoffs getting lost or agents not acknowledging work"
---

# Inter-Agent Handoff Standard (sessions_send)

Use this standard for **all** agent-to-agent requests in OpenClaw teams.

Telegram bots cannot reliably read each other, so handoffs must happen via `sessions_send`.

## Rules

1. Every handoff must include the required fields below.
2. Receiver must ACK quickly in the shared topic.
3. Receiver must post a completion update in the shared topic.
4. If blocked or overdue, receiver escalates with a blocker report.
5. No free-form handoffs for production workflows.
6. Receivers post lifecycle messages (ACK / DONE / BLOCKED) **explicitly via the `message` tool** targeting the shared topic — not by reply or by relying on the target session's stored channel state. See the platform note below.

## Platform Note: `sessions_send` Channel State

`sessions_send` is the spine of bot-to-bot handoffs, but it has a known issue ([openclaw/openclaw#31671](https://github.com/openclaw/openclaw/issues/31671)) where calling it from one agent into another's session can rewrite the target session's stored `channel` from `telegram` to `webchat`. The target agent can then stop receiving subsequent Telegram messages from the human.

Until this is fixed upstream:

- Treat `sessions_send` as a one-way trigger, not a routing rewrite.
- Always post ACK / DONE / BLOCKED by calling the `message` tool with an explicit `channel: "telegram"` and the `deliver_to` topic — do not assume the receiver's session knows where to reply.
- If you notice a teammate has gone quiet in Telegram after a handoff, suspect the channel-state corruption and have the human re-pair / resend in the topic to re-anchor the session.

## Required Handoff Format

Send this exact structure in `sessions_send`:

```text
HANDOFF
from: <agent-id>
to: <agent-id>
task_id: <short-unique-id>
priority: P0|P1|P2
summary: <one-line task summary>
context: <facts, links, paths, branch, constraints>
deliver_to: telegram:<group-id>:<topic-id>
deadline: <ISO timestamp or "asap">
max_retries: <integer, optional — failure budget before auto-escalation>
done_when:
- <acceptance criterion 1>
- <acceptance criterion 2>
```

`max_retries` is the receiver's retry budget for this handoff. If they hit the cap without satisfying `done_when`, they escalate (BLOCKED) instead of looping. Default 3 if unspecified.

## ACK Standard (Receiver)

Post this in the target topic within 2 minutes:

```text
ACK <task_id> — accepted
ETA: <time estimate>
```

If rejecting:

```text
ACK <task_id> — rejected
Reason: <clear reason>
Need: <what is required to proceed>
```

## Progress Preambles (Receiver, optional)

For tasks that take more than a few minutes, post short progress preambles between ACK and DONE so the topic doesn't go silent. One sentence, ~10 words, in the topic:

```text
<task_id>: explored the auth module; now patching the refresh logic.
<task_id>: tests passing locally; running CI before posting DONE.
```

Skip preambles for tasks that finish in under ~2 minutes — ACK→DONE is fine.

## Completion Standard (Receiver)

Before posting DONE, run the pre-DONE check:

- Every location named in `done_when` is actually updated.
- All required verifications (lint, tests, build) ran and passed.
- No unresolved TODOs introduced by this work.
- The diff is minimal — no stray refactors, no scope creep.

If any check fails, do not post DONE — finish the gap or post BLOCKED.

Then post:

```text
DONE <task_id>
Result: <what was delivered>
Evidence:
- <link/path/screenshot>
- <link/path/screenshot>
Risks/Notes: <optional — unrelated issues spotted, follow-up tasks, formatting caveats>
```

## Escalation Standard

If blocked >15 minutes or deadline risked:

```text
BLOCKED <task_id>
Blocker: <what is blocked>
Impact: <what cannot proceed>
Need decision from: <agent-id or human>
Options:
1) <option + tradeoff>
2) <option + tradeoff>
Recommendation: <best option>
```

## Example

```text
HANDOFF
from: coder
to: qa
task_id: build-142
priority: P1
summary: Validate checkout fix for coupon rounding bug
context: branch=fix/coupon-rounding, env=staging, PR=#142
deliver_to: telegram:-1001234567890:13
deadline: 2026-03-08T10:00:00+02:00
done_when:
- Repro no longer fails on staging
- Regression checks for totals pass
```
