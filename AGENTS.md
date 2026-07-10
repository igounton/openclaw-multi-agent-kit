# AGENTS.md

READ ./.agents/AGENTS.base.md BEFORE ANYTHING (skip if missing).

READ ./AGENTS.local.md AFTER this file if present (machine-only overrides, git-ignored).

Work style: telegraph; noun-phrases ok; drop grammar; min tokens.
Repo-specific hard rules only. Shared rules (Reviews, PR/CI, Git, Runtime Safety, Workflows) live in AGENTS.base.md.

## Core
- Template/docs-only repo — nothing executes; everything is markdown/JSON/JSONC copied into OpenClaw workspaces
- Public repo — no secrets, tokens, internal IPs, or machine-specific paths in tracked files
- docs/architecture.md carries the structure tree + platform concepts — update it whenever files, structure, or config keys change (was CLAUDE.md's job)
- Canonical OpenClaw docs = docs.openclaw.ai — verify config keys against the live reference before editing schema-related templates

## Project Defaults (repo-specific)
- Model-agnostic: never pin model ids in templates or example configs; agents inherit the runtime default
- YAML frontmatter at line 1 of every SOUL.md / SKILL.md — never prepend H1s or blockquotes above it
- Placeholders stay placeholders: `[Your ... Name]`, `YOUR_*`, `{WORKSPACE}` — never real values
- `examples/full-team.json` (10 core agents) and `examples/minimal-team.json` (3) must stay in sync, validate as JSON, and omit `model` fields
- Multi-agent topics: ALL bots `requireMention: true`; orchestrator `enabled: false` on topics owned by others
- Cross-agent flows follow docs/inter-agent-handoff-standard.md (HANDOFF/ACK/DONE/BLOCKED)
- Skills format: YAML frontmatter (`name`, `description`, `version`) + markdown body with `## Install`
- Editing checklists (what else to touch when adding agents/docs/skills/config keys): docs/architecture.md § Editing guidelines
- INSTRUCTIONS.md = single source of truth for the AI-readable setup flow

## Workflows (repo-specific)
- No release pipeline (docs-only kit) — no `release` command; curate CHANGELOG per landed series via `changelog`
