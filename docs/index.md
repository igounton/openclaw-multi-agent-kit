---
summary: "Index of all docs — routing data for agents"
read_when:
  - "Starting a new session — what docs exist?"
  - "Deciding which doc to read for a task"
---

# docs/ Index

Every doc carries YAML frontmatter (`summary:` + `read_when:`). Agents read this index to know what exists and when to read it. Setup flow itself lives in the root [INSTRUCTIONS.md](../INSTRUCTIONS.md).

## Orientation

| Doc | Summary |
|---|---|
| [architecture.md](architecture.md) | Repo structure, platform concepts, editing checklists |
| [agent-design-patterns.md](agent-design-patterns.md) | How to write effective SOUL.md files |
| [skills-system.md](skills-system.md) | Native Skills system, ClawHub, custom skills |

## Telegram & routing

| Doc | Summary |
|---|---|
| [supergroup-setup.md](supergroup-setup.md) | Step-by-step supergroup setup |
| [telegram-channel-architecture.md](telegram-channel-architecture.md) | Production lane design for topic routing |
| [telegram-dm-topics.md](telegram-dm-topics.md) | Forum topics inside a direct chat + ACP binding |
| [acpx-telegram.md](acpx-telegram.md) | ACPX coding-agent backend for Telegram |

## Operations

| Doc | Summary |
|---|---|
| [advanced-openclaw-practices.md](advanced-openclaw-practices.md) | 13 production operating patterns |
| [inter-agent-handoff-standard.md](inter-agent-handoff-standard.md) | HANDOFF/ACK/DONE/BLOCKED contract |
| [scaling.md](scaling.md) | When to add agents, cost tiers, circular triggers |
