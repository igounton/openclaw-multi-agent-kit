---
description: "Package current state for the next agent."
argument-hint: "[optional focus area]"
---
Read AGENTS.md. Produce a concise handoff for the next agent, in order:
1) Scope/status — what you were doing, done, pending, blockers
2) Working tree — summarize !`git status -sb` and unpushed commits
3) Branch/PR — current branch, PR number/URL, CI status
4) Running processes — background sessions/servers with attach commands
5) Tests/checks — what ran, results, what still needs running
6) Next steps — ordered bullets
7) Risks/gotchas — flaky tests, creds needed, brittle areas
Focus: $ARGUMENTS
