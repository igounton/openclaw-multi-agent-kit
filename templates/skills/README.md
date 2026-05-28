# SKILL.md Templates for Multi-Agent Teams

Use these as starter Skill packages when an agent needs deterministic workflows, strict formats, or reusable scripts.

## Included Templates

- `coding-handoff/SKILL.md` — branch/PR handoffs, review loops, release-ready status
- `research-intel/SKILL.md` — market scans, signal extraction, confidence scoring
- `leadgen-qualification/SKILL.md` — lead sourcing, enrichment, ICP scoring, next actions
- `content-repurpose/SKILL.md` — turn one source into X/LinkedIn/Reddit variants
- `ops-triage/SKILL.md` — inbox/calendar/task triage with priority routing
- `telegram-topic-setup/SKILL.md` — automated Telegram topic creation and agent binding
- `acpx-session/SKILL.md` — ACPX session management for agent-to-coding-agent delegation
- `template-audit/SKILL.md` — audit SOUL/SKILL/IDENTITY templates for frontmatter, duplicates, descriptions, placeholders, and prompt-budget size

## Usage

1. Copy one template into an agent-local skills folder.
2. Rename `name`/`description` in frontmatter.
3. Add role-specific references under `references/`.
4. Keep SKILL.md concise; move long docs to references.

Recommended path:

`agents/<agent>/skills/<skill-name>/SKILL.md`

## Native Skills System

OpenClaw v2026.3.24+ includes a built-in skills system with ClawHub marketplace, Control UI management, and one-click install. See `docs/skills-system.md` for full documentation.

## ClawHub Compatibility

All templates in this directory use YAML frontmatter (`name`, `description`, `version`) compatible with the OpenClaw native skills system. Templates can be copied manually into agent skill folders or adapted for ClawHub publishing.
