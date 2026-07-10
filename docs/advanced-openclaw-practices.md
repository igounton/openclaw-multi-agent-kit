---
summary: "13 production-tested operating patterns for multi-agent OpenClaw teams"
read_when:
  - "Designing how a team operates day-to-day"
  - "Agents misrouting, looping, or drowning in noise"
---

# Advanced OpenClaw Practices (That Actually Work)

This is a pragmatic playbook for operating multi-agent teams in production.

## 1) Deterministic Routing First

Use explicit bindings for critical routes; keep wildcard fallbacks last.

Why: routing ambiguity is the #1 hidden source of "wrong agent answered" incidents.

Reference: OpenClaw multi-agent routing and binding precedence.

## 2) Topic Ownership + Secondary Responders

Treat each topic as a workflow lane:

- Primary agent owns throughput and final output quality.
- Secondary specialists are triggered via `sessions_send` for depth work.

Why: this balances speed and specialization without context fragmentation.

## 3) Strict Handoff Contract (ACK / DONE / BLOCKED)

Never do free-form handoffs in production pipelines.

Minimum contract:

- `ACK` within short SLA
- `DONE` with delivery location + outcome summary
- `BLOCKED` with required unblock input

Why: prevents silent dead-ends and makes orchestration measurable.

## 4) Session Scope Discipline

Use thread-bound/persistent sessions for long coding or research runs.
Use one-shot runs for isolated tasks.

Why: prevents context bloat while preserving continuity where needed.

## 5) Two-Speed Model Policy

Default model handles triage, scraping, and formatting.
Premium model handles synthesis, architecture, and high-risk decisions.

Why: strongest quality/cost ratio in live teams.

## 6) Memory Distillation Cadence

Daily: append raw operational memory.
Weekly: distill into durable playbooks and decision logs.

Why: avoids memory rot and token-heavy recall.

## 7) Escalation Budgets

Set max retries per workflow stage, then auto-escalate.

Why: kills infinite retry loops and hidden stalls.

## 8) Safety-by-Default Tool Policies

Restrict risky tools per agent role (write/delete/exec/network actions).
Expand only when the workflow proves stable.

Why: limits blast radius during early iterations.

## 9) Operational SLOs for Agents

Track per-lane metrics:

- ACK latency
- Completion latency
- Blocked rate
- Rework rate

Why: agent teams improve when measured like real systems.

## 10) Final Output Schema

Every substantial task should end with:

- Context
- Decision
- Result
- Next Action

Why: this makes downstream handoffs and human review dramatically easier.

## 11) Use Skills for Deterministic Workflows

When an agent needs to follow a strict, repeatable process (scoring leads, triaging inbox, qualifying research signals), wrap it in a SKILL.md. Skills give you a single place to update the workflow, enforce output schemas, and add acceptance criteria. They are loaded by the agent at runtime and produce consistent results regardless of the agent's personality. See [docs/skills-system.md](skills-system.md) for the native Skills system.

## 12) ACPX for Coding Subagent Delegation

Don't have agents write code via prompt engineering — use ACPX to delegate to a dedicated coding agent (Claude Code, Codex, OpenCode). Define the coder as an ACP agent in openclaw.json with `runtime.type: "acp"`. Other agents trigger it via `sessions_spawn(runtime="acp")` from a subagent session. The coding agent gets its own workspace, tools, and session state. This separates coordination logic from implementation. See [docs/acpx-telegram.md](acpx-telegram.md).

## 13) Thread Bindings for Persistent Coding Sessions

Enable `channels.telegram.threadBindings` with `spawnSessions: true` (channel-level), optionally overriding per-account on the coder's Telegram account. This creates persistent ACP sessions per topic thread, so the coding agent remembers context across messages without re-establishing state. Set `idleHours` (48 recommended) to control session lifetime. Combine with the `/acp spawn <agent> --thread here` pattern for ad-hoc coding topics (Telegram uses `--thread`, not `--bind`).
