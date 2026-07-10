# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added
- Agent repo conventions: `AGENTS.md` hard rules, cross-agent slash commands in `.agents/commands/` (handoff/pickup/commit/fix/changelog), docs frontmatter + `docs/index.md`, and this changelog
- `docs/architecture.md`: repo structure, platform concepts, and editing checklists (moved from CLAUDE.md)

### Changed
- `CLAUDE.md` reduced to a one-line pointer to `AGENTS.md` (no duplicated rules)

## Backfill (pre-changelog history)

- `template-audit` skill for SOUL/SKILL/IDENTITY hygiene
- Skills System + ACPX Telegram integration docs and templates; config synced with docs.openclaw.ai
- Extended soul library (106 templates across 12 categories), upgraded Telegram channel architecture, skill templates
- Telegram DM forum topics guide with icons, ACP binding, thread setup; clarified the 3 routing models
- Mandatory inter-agent handoff standard (HANDOFF/ACK/DONE/BLOCKED over `sessions_send`)
- `requireMention: true` fix for all agents in multi-agent topics; kit-wide consistency fixes
- Initial kit: 10 agent templates, Telegram supergroup setup, AI-readable INSTRUCTIONS.md, MIT license
