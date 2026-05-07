---
name: carly-code-review
description: "Comprehensive code review using 7 parallel specialized sub-agents (correctness, security, performance, simplicity, UX, codebase integration, documentation). Synthesizes findings into a prioritized report with de-duplication and false positive filtering. Use when reviewing code changes, pull requests, or local diffs. Supports local git diff (no arguments) or GitHub PR (pass PR number or URL as argument)."
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
2. Read the full PR conversation — reviews, comments, and review comment replies:
   ```bash
   gh pr view $ARGUMENTS --json reviews,comments
   ```
   This gives you context on which prior suggestions were accepted, rejected, or discussed. Do not re-raise findings that the author has already responded to with a justification.
3. Get the PR commits and find which were pushed after that timestamp:
   ```bash
   gh pr view $ARGUMENTS --json commits
   ```
4. Diff only the new commits: `git diff <last-reviewed-commit-sha>..<latest-commit-sha>`
5. **Only review the new diff**, not the full PR
6. Start the report with: `**Incremental review** — reviewing changes since last review (<timestamp>).`

If there are no new commits since the last review, say so and stop.

If the diff is empty, tell the user and stop.

## Step 2 — Triage: select relevant reviewers

Not every diff needs all 7 reviewers. Before spawning agents, look at what the diff actually touches and skip reviewers that clearly don't apply.

**Always run:** correctness-reviewer, simplicity-reviewer, documentation-reviewer.

**Skip when not relevant:**

| Reviewer | Skip when... |
|---|---|
| **security-reviewer** | Changes are purely cosmetic (CSS, copy, formatting), test-only, or documentation-only |
| **performance-reviewer** | Changes are UI-only (templates, styles, static copy) or documentation/config-only with no logic changes |
| **ux-reviewer** | Changes are backend-only with no user-facing API, error message, or UI changes |
| **integration-reviewer** | Changes are test-only or documentation-only. **Do not skip** for new files — new code should still follow existing codebase conventions and patterns. |

**Trivial changes:** For clearly trivial diffs (typo fix, copy change, dependency bump), skip sub-agents entirely and review directly. State that a full review was not warranted and why.

Note which reviewers were skipped and why in the report summary.

## Step 3 — Assess project context

Before delegating, determine what kind of review rigor is appropriate. Project context is **two independent axes** — they're not a single scale. A small internal compliance tool can be high-consequence and low-scale; a free consumer game can be high-scale and low-consequence. Each axis tunes a different set of reviewers.

**Look for explicit signals first.** Check CLAUDE.md and README.md for stated context (e.g., "pre-launch," "early customers," "handles PII," "SOC 2," "internal tool," "consumer-facing at scale"). An explicit statement from the author overrides anything you'd infer from the code.

**If no explicit signal exists**, infer from PR description, commit messages, and repo signals (CI, tests, infra config, dependency footprint, domain hints in the code). Be conservative — modern tooling like CI and tests is cheap and present in most projects regardless of maturity, so it's a weak signal on its own.

### Axis 1 — Scale (tunes performance, simplicity, architecture)

| Scale | Signals | Calibration |
|---|---|---|
| **Pre-launch** | No real users yet — prototype, POC, MVP, solo build, "experiment." | Drop speculative scale concerns. Downgrade performance and architectural Warnings to Suggestions. |
| **Early users** | Small number of real users (handful of customers, beta, internal pilot). Project still evolving. | Be measured on performance and architecture. Flag clear regressions; don't optimize prematurely. |
| **At scale** | Many users, meaningful traffic or data volume, hot paths matter. | Full performance and architectural rigor. Promote performance Suggestions to Warnings where regressions are concrete. |

### Axis 2 — Consequence (tunes correctness, security, UX)

| Consequence | Signals | Calibration |
|---|---|---|
| **Low** | Failure is recoverable and low-impact — internal tool, hobby project, easily-rerun batch job, nothing sensitive. | Standard correctness/security; don't dwell on edge-case hardening. |
| **Standard** | Normal product code — failures cost user trust or time but aren't catastrophic. | Standard review across correctness, security, UX. |
| **High** | Failures have real consequences — handles money, PII/PHI, auth, compliance (SOC 2, HIPAA, GDPR), safety, or other irreversible effects. | Full rigor on correctness and security. Promote security and correctness Suggestions to Warnings where realistic. Take edge cases seriously. |

When ambiguous, default to **Early users / Standard**. Note explicitly in the report that you inferred the context, and suggest the author add a one-line marker to CLAUDE.md so future reviews don't have to guess:

```
Project context: scale=early-users, consequence=high  # handles customer payment data
```

Pass both axes to each sub-agent so they can calibrate their own severity assessments.

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
3. **Recalibrate severity.** Apply both project-context axes from Step 3. Use **scale** to tune performance/architecture findings (downgrade for pre-launch, promote for at-scale) and **consequence** to tune correctness/security findings (be lenient for low-consequence, promote Suggestions to Warnings for high-consequence). The two axes are independent — a high-consequence pre-launch tool still warrants strict security review even though performance findings should be soft.
4. **De-duplicate.** If multiple agents flagged the same issue, keep the most specific version.
5. **Drop downstream noise.** If fixing a Critical would resolve a Suggestion, drop the Suggestion.
6. **Prioritize.** Critical → Warning → Suggestion. Within each severity, group by file.

A report with zero findings is a valid outcome. A short report with only high-impact findings is better than a long report that wastes the author's time.

## Step 6 — Report

```markdown
# Code Review Report

## Summary
**Project context:** scale=[pre-launch / early-users / at-scale], consequence=[low / standard / high] — [explicit from CLAUDE.md/README, or inferred from <signal>; if inferred, suggest adding a `Project context:` line to CLAUDE.md]
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

### Tool usage notes

- Use the **Grep** and **Read** tools to search and read files — do not shell out to `grep`, `cat`, or `find` via Bash.
- Only use **Bash** for `git` and `gh` commands.
- **Do not chain commands** with `&&` or `;` — run each `git` or `gh` command as a separate Bash call.
