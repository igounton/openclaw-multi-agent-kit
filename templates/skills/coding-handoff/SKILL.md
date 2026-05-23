---
name: coding-handoff
description: Standardize coding handoffs between builder, QA, and deploy agents. Use when a coding task moves across stages and requires strict ACK/DONE/BLOCKED status updates, branch metadata, and PR readiness checks.
version: "1.0.0"
---

## Install

Copy to `agents/<coder>/skills/coding-handoff/SKILL.md`. Requires agents configured with `sessions_send` for inter-agent handoffs.

# Coding Handoff Skill

## Required Handoff Envelope

Always send handoff messages using:

- `from`
- `to`
- `task_id`
- `priority` (P1/P2/P3)
- `summary`
- `context` (branch, commit, environment)
- `done_when` (checklist)
- `deliver_to` (topic/channel target)

## Lifecycle

1. Post `ACK` quickly with ownership and ETA.
2. Execute scoped checks.
3. Post `DONE` with evidence (tests, logs, diff summary) or `BLOCKED` with explicit unblock requirements.

## PR Readiness Gate

Before DONE, verify:

- lint/type/test status
- regression risk notes
- rollback note
- unresolved TODOs list

If any gate fails, return BLOCKED, not DONE.

## Branch Naming

Branches created during a coding handoff should encode origin and task for traceability:

```
<agent-id>/<task_id>-<short-slug>
```

Examples: `coder/build-142-coupon-rounding`, `coder/auth-jwt-001-refresh-token`.

This pattern makes it easy to grep history for an agent's work, link branches back to task IDs in topic messages, and reason about cleanup. Avoid bare slugs (`fix-bug`) or timestamp-only branches — they don't carry provenance.

Do not commit or push unless the task explicitly asks for it. The DONE message references the local branch; pushing/PR-opening is a separate explicit step.

## Scope Discipline

A handoff is bounded by `done_when`. Anything outside that scope:

- **Unrelated bug spotted while working:** mention it in the DONE message under `Risks/Notes`. Do not fix it in the same change.
- **Adjacent refactor that "would be nice":** skip. File a follow-up task instead.
- **Test that fails for unrelated reasons:** flag it, route around it, don't paper over it.

Scope creep turns a 1-hour handoff into a 6-hour mystery. The handoff format exists to keep work bounded — respect it.
