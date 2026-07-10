---
description: "Safe commit with Conventional Commits."
argument-hint: "[scope/message hint]"
---
Prefer the guardrail script (clears index, stages only listed paths, rejects `.`/dirs/globs):
`sh .agents/skills/committer/scripts/committer.sh "<type>: <message>" <file> [<file> ...]`
(in the standards repo itself, the path is `skills/committer/scripts/committer.sh`)
Fallback (script unavailable): stage only explicit paths (never `git add .`), review !`git diff --staged`.
Write a Conventional Commit (feat|fix|refactor|build|ci|chore|docs|style|perf|test).
Do not push unless asked.
Hint: $ARGUMENTS
