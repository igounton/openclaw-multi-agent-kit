---
name: template-audit
description: Audit SOUL.md, SKILL.md, IDENTITY.md, and workspace templates for frontmatter compliance, duplicate names across core/extended catalogs, description quality, placeholder hygiene, and prompt-budget size before they are copied into an OpenClaw workspace.
version: "1.0.0"
---

## Install

Copy to `agents/<maintainer>/skills/template-audit/SKILL.md` for an orchestrator or maintainer agent that owns workspace setup. Run against any directory that mirrors the layout of `openclaw-multi-agent-kit/templates/` — a kit checkout, a vendored copy, or an in-progress agent workspace.

# Template Audit Skill

Use this when picking which templates to copy into a new workspace, when reviewing a PR that adds or edits templates, or when team prompt size feels heavy and you need to know which SOUL/SKILL files dominate the budget.

## When to Use

- Adding a new agent → check the target soul template before copying.
- Adding a new skill → verify frontmatter, install path, and that no near-duplicate exists.
- Workspace bloat → identify the heaviest soul/skill templates by byte size.
- Cross-catalog overlap → confirm whether a core template (e.g. `coding-agent.md`) already covers what an extended template offers (e.g. `engineering/*`).

## Audit Checklist

Run these passes against the target directory tree. Report findings; do not edit files in-place unless the user asks.

### 1. Frontmatter at line 1

For every `.md` file in `templates/soul/**`, `templates/skills/**/SKILL.md`, and `templates/identity/*.md`:

- The first non-empty line must be `---`.
- No H1 header, blockquote, or comment above the YAML block.
- Required fields:
  - SOUL.md: `name`, `description` (no `version` required by convention).
  - SKILL.md: `name`, `description`, `version`.
  - IDENTITY.md: free-form; flag only if the file looks templated but missing fields.

Flag any file that fails — these break OpenClaw's loader silently.

### 2. Duplicate / near-duplicate names

Scan for:

- Same `name` across core (`templates/soul/*.md`) and extended (`templates/soul/<category>/*.md`).
- Same `name` across two extended categories.
- Same `name` across `templates/skills/*/SKILL.md`.

For each collision report: which file would win, which one should be merged or renamed, and whether bodies overlap >60% (rough word-set similarity is fine — exact diff not required).

### 3. Description quality

For each template, flag descriptions that:

- Exceed ~200 characters → suggest a tighter rewrite, preserving trigger nouns (product, tool, action, object).
- Use vague phrasing ("helps with", "is responsible for") instead of concrete triggers ("Use when ...").
- Are missing entirely.

Preserve trigger nouns — they are how downstream agents and routers find the right template.

### 4. Placeholder hygiene

Per repo convention, templates must contain replaceable placeholders before they are copied. Confirm at least one of: `[Your ... Name]`, `YOUR_*`, `[Name]`, `{WORKSPACE}`. Flag templates that are fully filled in with example values — they will look "done" and get copied without customization.

### 5. Extended catalog adaptation footer

Every file under `templates/soul/<category>/*.md` should carry an "OpenClaw Adaptation Notes" footer (per the kit's own README). Flag any extended template missing it — that footer is what tells the copier the body is generic agency prose and needs rewriting.

### 6. Prompt-budget size

For each template, report:

- byte size
- estimated tokens (`ceil(utf8_bytes / 4)`)

Sort soul templates by size descending. Flag any soul file >8k tokens — these will dominate an agent's system prompt. Flag any skill file >2k tokens — skills should stay tight.

## Output Format

Group findings into sections matching the checklist:

```
## Template Audit Report

### Frontmatter issues
- templates/soul/<file>: <issue>

### Duplicates
- <name> appears in: <paths> — recommend: <action>

### Description candidates
- <path>: <current length> chars — <suggested rewrite or summary of what to tighten>

### Placeholder issues
- <path>: <missing placeholder type>

### Adaptation footer missing
- templates/soul/<category>/<file>

### Size leaders
1. <path> — <tokens> tokens
2. ...
```

End with a one-paragraph summary: total templates scanned, total flagged, top suggested cleanup actions.

## Output Policy

- Suggest first; edit only when the user asks.
- If asked to apply cleanup, make small grouped commits: frontmatter fixes, description rewrites, deletes, adaptation footers.
- Never delete an extended template just because a core template covers the same role — extended catalogs are starters, and the user may want them as alternatives. Flag the overlap and let the user decide.
- Preserve placeholders. A "cleanup" that resolves `[Your Company Name]` to a real value is not cleanup, it is contamination of the template.

## Heuristics

- Core vs extended overlap is expected (core `coding-agent` ≈ `engineering/backend-architect`). Report it, do not auto-resolve it.
- Near-identical SKILL bodies across two skill dirs are a real duplicate. Suggest keeping the one with the cleaner install path and richer references.
- Workspace files (`templates/workspace/*.md`) have no frontmatter by design — skip the frontmatter pass for that subtree.
- Token estimate is heuristic; do not gate decisions on a 5% difference. Use it to find outliers, not to rank similar-sized files.
