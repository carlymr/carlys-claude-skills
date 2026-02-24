---
name: carly-code-review
description: "Comprehensive code review using 7 parallel specialized sub-agents (correctness, security, performance, simplicity, UX, codebase integration, documentation). Synthesizes findings into a prioritized report with de-duplication and false positive filtering. Use when reviewing code changes, pull requests, or local diffs. Supports local git diff (no arguments) or GitHub PR (pass PR number or URL as argument)."
disable-model-invocation: true
argument-hint: "[PR number or URL] (optional, defaults to local diff)"
allowed-tools: Bash(git *), Bash(gh *), Read, Write, Grep, Glob, Task
---

# Code Review Orchestrator

Coordinate a code review by delegating to specialized reviewer sub-agents in parallel, then synthesize their findings into a single actionable report.

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

**Incremental review (PR update)?** Before fetching the full diff, check if this PR already has a prior review from a previous run. Reviews posted via `gh pr review` appear under `reviews`, not `comments`:
```bash
gh pr view $ARGUMENTS --json reviews --jq '.reviews[] | select(.body | startswith("# Code Review Report")) | {createdAt: .submittedAt}'
```
If a prior review exists:

1. Note the `submittedAt` timestamp of the most recent review
2. Get the PR commits and find which were pushed after that timestamp:
   ```bash
   gh pr view $ARGUMENTS --json commits
   ```
3. Diff only the new commits: `git diff <last-reviewed-commit-sha>..<latest-commit-sha>`
4. **Only review the new diff**, not the full PR
5. Start the report with: `**Incremental review** — reviewing changes since last review (<timestamp>).`

If there are no new commits since the last review, say so and stop.

If the diff is empty, tell the user and stop.

## Step 2 — Triage: select relevant reviewers

Not every diff needs all 7 reviewers. Before spawning agents, look at what the diff actually touches and skip reviewers that clearly don't apply.

**Always run:** correctness-reviewer, documentation-reviewer.

**Skip when not relevant:**

| Reviewer | Skip when... |
|---|---|
| **security-reviewer** | Changes are purely cosmetic (CSS, copy, formatting), test-only, or documentation-only |
| **performance-reviewer** | Changes are UI-only (templates, styles, static copy) or documentation/config-only with no logic changes |
| **simplicity-reviewer** | Changes are test-only, documentation-only, or config-only |
| **ux-reviewer** | Changes are backend-only with no user-facing API, error message, or UI changes |
| **integration-reviewer** | Changes are test-only or documentation-only. **Do not skip** for new files — new code should still follow existing codebase conventions and patterns. |

**Trivial changes:** For clearly trivial diffs (typo fix, copy change, dependency bump), skip sub-agents entirely and review directly. State that a full review was not warranted and why.

Note which reviewers were skipped and why in the report summary.

## Step 3 — Assess project context

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

## Step 4 — Delegate to selected reviewers in parallel

Launch the selected reviewer sub-agents (from Step 2) in a **single message** using the Task tool. Pass each one the full diff, changed file list, PR description (if available), and the project context classification from Step 3.

The 7 reviewer agents are:
1. **correctness-reviewer** — bugs, logic errors, edge cases
2. **security-reviewer** — vulnerabilities, injection, auth, secrets
3. **performance-reviewer** — complexity, N+1, allocations, caching
4. **simplicity-reviewer** — over-engineering, unnecessary abstractions
5. **ux-reviewer** — confusing APIs, error messages, accessibility
6. **integration-reviewer** — duplicated functionality, pattern mismatches
7. **documentation-reviewer** — CLAUDE.md adherence, outdated READMEs, stale docs

Each agent returns findings in this format:
```
**[SEVERITY]** `file/path.ext:L<start>-L<end>` — [title]
[description]
**Suggested fix:** [fix]
```

## Step 5 — Synthesize

**Bias for simplicity.** Every suggestion you include has a cost: code becomes harder to understand, the author spends time on revisions, and reviewers spend cognitive effort evaluating changes. Only include findings where the benefit clearly outweighs these costs. When in doubt, leave it out.

1. **Verify.** For each Critical/Warning finding, read the file yourself with Read. Check whether the existing code already handles the concern (e.g., a framework guard, a runtime guarantee, an upstream validation). If the concern is already addressed, drop it entirely — do not include it with a note that it's "already handled."
2. **Apply the cost/benefit test.** For each remaining finding, ask: is the risk realistic and significant enough to justify the cost of changing the code? Drop findings where:
   - The scenario is theoretically possible but extremely unlikely in practice
   - The fix adds complexity (error handling, validation, abstractions) that makes the code harder to read and maintain
   - The finding is defensive programming against a situation that the system's architecture already prevents
   - The suggestion is "nice to have" hardening with no concrete failure scenario
3. **Recalibrate severity.** Apply the project context from Step 3. For a prototype, downgrade performance and simplicity Warnings to Suggestions. For production code, consider promoting security and performance Suggestions to Warnings.
4. **De-duplicate.** If multiple agents flagged the same issue, keep the most specific version.
5. **Drop downstream noise.** If fixing a Critical would resolve a Suggestion, drop the Suggestion.
6. **Prioritize.** Critical → Warning → Suggestion. Within each severity, group by file.

A report with zero findings is a valid outcome. A short report with only high-impact findings is better than a long report that wastes the author's time.

## Step 6 — Report

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

**Always** end the report with this attribution footer:

```markdown
---
🤖 *Generated by [carly-code-review](https://github.com/carlymr/carlys-claude-skills) via Claude Code*
```

### PR comments

When reviewing a **GitHub PR**, also post the findings to the PR itself.

**Important:** Use the Write tool to write the report (including the attribution footer) to a temp file, then pass it via `--body-file` to avoid shell escaping issues:
```bash
# After writing the report to /tmp/review-report.md with the Write tool:
gh pr review $PR_NUMBER --comment --body-file /tmp/review-report.md
```

**Inline diff comments** — post Critical and Warning findings as inline comments on the specific lines. Write each comment body to a temp file with the Write tool, then post:
```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments \
  --method POST \
  -f path="<file>" \
  -f line=<line_number> \
  -f side="RIGHT" \
  --body-file /tmp/inline-comment.md
```

If inline commenting is denied by permissions, do not retry — the summary review comment already contains all findings.

Do not post Suggestions as inline comments — those go only in the summary review comment.

### Tool usage notes

- Use the **Grep** and **Read** tools to search and read files — do not shell out to `grep`, `cat`, or `find` via Bash.
- Only use **Bash** for `git` and `gh` commands.
- **Do not chain commands** with `&&` or `;` — run each `git` or `gh` command as a separate Bash call.
