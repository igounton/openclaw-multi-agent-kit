---
description: "Curate CHANGELOG.md Unreleased section from commits since the last tag."
argument-hint: "[optional baseline tag]"
---
Curate the user-facing changes since the last release into `## [Unreleased]`.
This is the input to `release` — run it before, not during, a release.

1) Pick baseline: $1 if given, else !`git describe --tags --abbrev=0 2>/dev/null || echo "(no tags — use full history)"`
2) Collect commits since baseline: review `git log <baseline>..HEAD --oneline --reverse`. Skim diffs where impact is unclear.
3) Curate user-facing entries only:
   - Include: shipped features, fixes, breaking changes, notable UX/behavior changes.
   - Exclude: internal refactors, typo-only edits, dep bumps with no user impact, anything added then removed in the same window.
   - Order by impact: breaking → features → fixes → misc.
   - Add `#123` PR/issue refs when known; never paste raw commit hashes.
4) Edit `CHANGELOG.md`: ensure `## [Unreleased]` exists at the top; append bullets under it, matching existing style (one bullet per entry, one line, past-tense, code in backticks). Do not hard-wrap long bullets.
5) Sanity check: renders cleanly, no duplicates, concise wording. Do not commit unless asked.
Baseline hint: $ARGUMENTS
