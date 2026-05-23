# Scaling Your Team

Practical guidance for growing your agent team. For step-by-step setup instructions, see [INSTRUCTIONS.md](../INSTRUCTIONS.md).

## When to Add a New Agent

Add an agent when:
- A single agent's SOUL.md is covering two clearly distinct domains (e.g., "content creation AND community management")
- Tasks regularly queue up behind one agent, creating a bottleneck
- A domain requires specialized model selection (e.g., coding needs opus, but ops tasks waste money on it)

**Don't** add an agent when:
- You can solve the problem with a better prompt in an existing agent's SOUL.md
- The "new role" would only trigger a few times per week — give it to an existing agent as a secondary responsibility
- You're at 8+ agents and coordination overhead is already noticeable — more agents means more `sessions_send` chains to debug

## When to Split a Single-Agent Topic into a Team Topic

A topic should become a team topic when:
- The primary agent regularly needs a second opinion (e.g., coder needs QA)
- Workflow has a natural handoff (build -> test -> deploy)
- The human wants visibility into multi-step processes in one place

When you split, remember:
- ALL agents in the topic must switch to `requireMention: true`
- The orchestrator must have `enabled: false` for the new topic
- Update SUPERGROUP-MAP.md and each agent's SOUL.md team integration section

## Cost Considerations

Model selection is the biggest lever on monthly spend, but the templates in this repo are intentionally **model-agnostic** — agents inherit whatever model your OpenClaw runtime is configured with. Tune cost by overriding `model` on individual agents in `openclaw.json`, not by editing templates.

Rough tiers (use whichever your provider equivalents are):

| Tier | Best For |
|------|----------|
| **Most capable** (top-tier reasoning) | Complex coding, architecture decisions |
| **Balanced** (default tier) | Orchestration, research, content, lead gen |
| **Smallest / cheapest** | QA checks, ops tasks, community, triage |

Rules of thumb:
- Start every new agent on the runtime default and upgrade only if output quality is insufficient
- Orchestrator needs real reasoning — don't downgrade it below the balanced tier
- Coding agents benefit most from stronger models; the cost difference pays for itself in fewer QA cycles
- Pinning specific model ids in templates couples your team to one provider — leave model unset by default and override per-agent only when needed

## Avoiding Circular Triggers

Agent dependency chains can loop if not designed carefully:

**Bad:** Coder triggers QA -> QA finds issue -> QA triggers Coder -> Coder "fixes" -> triggers QA -> loop forever

**Good:** Design one-directional chains with human checkpoints:
- Coder -> QA -> DevOps (linear chain)
- QA fails -> Coder fixes -> Coder re-triggers QA (bounded retry, not infinite loop)
- Add a max-retry note in the agent's SOUL.md: "After 2 failed QA cycles, escalate to orchestrator"

**Watch for:**
- Research -> Growth -> Research feedback loops (break with: Growth writes to SIGNALS.md, Research reads on next cycle — not via `sessions_send`)
- Content -> Community -> Content cross-posting loops (break with: human approval gate between cycles)

## Adding an Agent — Quick Reference

1. Create bot via @BotFather, add to supergroup as admin
2. Create workspace: `~/.openclaw/workspace/agents/[name]/`
3. Copy and customize SOUL.md and IDENTITY.md from templates
4. Add to `agents.list`, `bindings`, and `channels.telegram.accounts` in openclaw.json
5. Update SUPERGROUP-MAP.md and AGENTS.md
6. Update teammate SOUL.md files to reference the new agent
7. `openclaw restart` and test

See [INSTRUCTIONS.md Phase 3-5](../INSTRUCTIONS.md) for the full walkthrough.
