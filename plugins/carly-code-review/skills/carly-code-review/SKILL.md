---
name: carly-code-review
description: "Comprehensive code review using 6 parallel specialized sub-agents (correctness, security, performance, simplicity, UX, codebase integration). Synthesizes findings into a prioritized report with de-duplication and false positive filtering. Use when reviewing code changes, pull requests, or local diffs. Supports local git diff (no arguments) or GitHub PR (pass PR number or URL as argument)."
disable-model-invocation: true
argument-hint: "[PR number or URL] (optional, defaults to local diff)"
allowed-tools: Bash(git *), Bash(gh *), Read, Grep, Glob, Task
---

# Code Review Orchestrator

Coordinate a code review by delegating to 6 specialized reviewer sub-agents in parallel, then synthesize their findings into a single actionable report.

## Step 1 — Get the diff

**No arguments?** Review local changes:
```bash
git diff
git diff --cached
git diff --name-only && git diff --cached --name-only
```

**PR number or URL passed in `$ARGUMENTS`?** Fetch from GitHub:
```bash
gh pr diff $ARGUMENTS
gh pr diff $ARGUMENTS --name-only
gh pr view $ARGUMENTS
```

If the diff is empty, tell the user and stop.

## Step 2 — Delegate to 6 reviewers in parallel

Launch all 6 reviewer sub-agents in a **single message** using the Task tool. Pass each one the full diff, changed file list, and PR description (if available).

The 6 reviewer agents are:
1. **correctness-reviewer** — bugs, logic errors, edge cases
2. **security-reviewer** — vulnerabilities, injection, auth, secrets
3. **performance-reviewer** — complexity, N+1, allocations, caching
4. **simplicity-reviewer** — over-engineering, unnecessary abstractions
5. **ux-reviewer** — confusing APIs, error messages, accessibility
6. **integration-reviewer** — duplicated functionality, pattern mismatches

Each agent returns findings in this format:
```
**[SEVERITY]** `file/path.ext:L<start>-L<end>` — [title]
[description]
**Suggested fix:** [fix]
```

## Step 3 — Synthesize

1. **Verify.** For each Critical/Warning finding, read the file yourself with Read. Drop false positives.
2. **De-duplicate.** If multiple agents flagged the same issue, keep the most specific version. Note which other areas also flagged it.
3. **Drop downstream noise.** If fixing a Critical would resolve a Suggestion, drop the Suggestion.
4. **Prioritize.** Critical → Warning → Suggestion. Within each severity, group by file.

## Step 4 — Report

```markdown
# Code Review Report

## Summary
[1-2 sentences: count of findings by severity, highlight the most important issues]

## Critical
### `file/path.ext:L10-L25` — [title]
[Description and why it matters]
**Suggested fix:** [approach]

## Warnings
### `file/path.ext:L42-L48` — [title]
[Description]
**Suggested fix:** [approach]

## Suggestions
### `file/path.ext:L100` — [title]
[Description]
**Suggested fix:** [approach]
```

Omit empty severity sections. If no findings at all, say so.

### PR inline comments

When reviewing a **GitHub PR**, also post inline comments for Critical and Warning findings:

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments \
  --method POST \
  -f path="<file>" \
  -f line=<line_number> \
  -f side="RIGHT" \
  -f body="**[SEVERITY]**: [description]

**Suggested fix:** [fix]"
```

Do not post Suggestions as PR comments — those go only in the conversation report.
