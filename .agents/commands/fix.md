---
description: "Run quality gates and fix all failures."
---
Run the repo's quality gates and fix every failure until green:
- JSON examples validate: !`python3 -m json.tool examples/full-team.json > /dev/null && python3 -m json.tool examples/minimal-team.json > /dev/null && echo JSON-OK`
- Docs front matter: `python3 .agents/skills/docs-list/scripts/docs-list.py`
- Template hygiene: SOUL.md/SKILL.md frontmatter at line 1; placeholders (`[Your ...]`, `YOUR_*`, `{WORKSPACE}`) not replaced with real values; no model ids pinned
- Examples in sync: full-team.json = 10 core agents, minimal-team.json = 3, both without `model` fields

Re-run until clean. Update docs/CHANGELOG for visible changes.
Confirm `git status -sb` clean and on the expected branch.
