---
name: carly-code-review
description: "Comprehensive code review using 6 parallel specialized sub-agents (correctness, security, performance, simplicity, UX, codebase integration). Synthesizes findings into a prioritized report with de-duplication and false positive filtering. Use when reviewing code changes, pull requests, or local diffs. Supports local git diff (no arguments) or GitHub PR (pass PR number or URL as argument)."
disable-model-invocation: true
argument-hint: "[PR number or URL] (optional, defaults to local diff)"
allowed-tools: Bash(git *), Bash(gh *), Read, Write, Grep, Glob, Task
---

# Code Review Orchestrator

Coordinate a code review by delegating to 6 specialized reviewer sub-agents in parallel, then synthesize their findings into a single actionable report.

## Step 1 — Get the diff

**No arguments?** Review local uncommitted and staged changes against the default branch:
```bash
# Detect the default branch (don't assume main)
git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'
# Diff current working tree against the default branch
git diff <default-branch>
git diff <default-branch> --name-only
```

**PR number or URL passed in `$ARGUMENTS`?** Fetch from GitHub:
```bash
gh pr diff $ARGUMENTS
gh pr diff $ARGUMENTS --name-only
gh pr view $ARGUMENTS
```

**Incremental review (PR update)?** Check if this PR already has a prior review comment from a previous run by looking for an existing comment starting with "# Code Review Report":
```bash
gh pr view $ARGUMENTS --comments --json comments
```
If a prior review exists, get only the new changes since that review. Use the timestamp of the prior review comment to find new commits:
```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/commits
```
Then diff only the new commits rather than the full PR diff. Mention in the report header that this is an incremental review of new changes since the last review.

If the diff is empty, tell the user and stop.

## Step 2 — Assess project context

Before delegating, determine the project's maturity and what kind of review rigor is appropriate. Check these signals (in parallel where possible):

- **CLAUDE.md** — Read it if it exists. Look for project stage, conventions, or quality expectations.
- **PR description / commit messages** — Look for keywords like "prototype," "POC," "hack week," "production," "migration," "v2," etc.
- **Repo signals** — Check for CI config, test directories, linting config, Docker/k8s files, dependency lock files. Their presence (or absence) indicates maturity.

Classify the project context as one of:

| Context | Signals | Review calibration |
|---|---|---|
| **Prototype / POC** | "proof of concept," "demo," "experiment," no CI, few tests, small repo | Focus on correctness and security. Downgrade performance, simplicity, and integration findings to Suggestion at most. |
| **Active development** | Feature branches, growing test suite, some CI | Standard review across all areas. |
| **Production / Enterprise** | Mature CI/CD, extensive tests, monitoring, infra config, "production" or "enterprise" in docs | Full rigor. Performance and security findings get extra weight. |

Pass this context classification to each sub-agent so they can calibrate their own severity assessments.

## Step 3 — Delegate to 6 reviewers in parallel

Launch all 6 reviewer sub-agents in a **single message** using the Task tool. Pass each one the full diff, changed file list, PR description (if available), and the project context classification from Step 2.

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

## Step 4 — Synthesize

1. **Verify.** For each Critical/Warning finding, read the file yourself with Read. Drop false positives.
2. **Recalibrate severity.** Apply the project context from Step 2. For a prototype, downgrade performance and simplicity Warnings to Suggestions. For production code, consider promoting security and performance Suggestions to Warnings.
3. **De-duplicate.** If multiple agents flagged the same issue, keep the most specific version. Note which other areas also flagged it.
4. **Drop downstream noise.** If fixing a Critical would resolve a Suggestion, drop the Suggestion.
5. **Prioritize.** Critical → Warning → Suggestion. Within each severity, group by file.

## Step 5 — Report

```markdown
# Code Review Report

## Summary
**Project context:** [Prototype / Active development / Production]
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

### PR comments

When reviewing a **GitHub PR**, also post the findings to the PR itself.

**Important:** Use the Write tool to write the report to a temp file, then pass it via `--body-file` to avoid shell escaping issues:
```bash
# After writing the report to /tmp/review-report.md with the Write tool:
gh pr review $PR_NUMBER --comment --body-file /tmp/review-report.md
```

**Inline comments** — post Critical and Warning findings as inline comments on the specific lines:
```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments \
  --method POST \
  -f path="<file>" \
  -f line=<line_number> \
  -f side="RIGHT" \
  -f body="**[SEVERITY]**: [description]. **Suggested fix:** [fix]"
```

Do not post Suggestions as inline comments — those go only in the summary review comment.

### Tool usage notes

- Use the **Grep** and **Read** tools to search and read files — do not shell out to `grep`, `cat`, or `find` via Bash.
- Only use **Bash** for `git` and `gh` commands.
