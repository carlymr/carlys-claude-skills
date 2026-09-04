# Carly's Claude Skills

A plugin marketplace for Claude Code, containing shareable skills and specialized sub-agents.

## Installation

Add the marketplace:

```
/plugin marketplace add carlymr/carlys-claude-skills
```

Install the plugin:

```
/plugin install carly-tools@carlys-claude-skills
```

## Available Skills

### carly-code-review

Comprehensive code review using 7 parallel specialized sub-agents:
- **Correctness** — bugs, logic errors, missing edge cases
- **Security** — vulnerabilities, injection, auth flaws, secrets exposure
- **Performance** — algorithmic complexity, N+1 queries, unnecessary allocations
- **Simplicity** — over-engineering, unnecessary abstractions
- **UX** — confusing APIs, poor error messages, accessibility
- **Codebase Integration** — duplicated functionality, pattern mismatches
- **Documentation** — CLAUDE.md adherence, outdated READMEs, stale inline docs, changelog gaps

The orchestrator assesses project context on two axes — scale (pre-launch vs at scale) and consequence (what failures cost) — to calibrate review rigor, then synthesizes all findings — verifying against actual code, de-duplicating, and dropping minor comments that would resolve when fixing larger issues.

Supports local git diff (no arguments) or GitHub PR review.

```
/carly-code-review
/carly-code-review 123
/carly-code-review https://github.com/org/repo/pull/123
```

Also works in CI — see [GitHub Action Setup](#github-action-setup) below.

### carly-product-req

Guided co-authoring of a lightweight product requirements doc — aimed at non-technical teammates (PMs, designers, founders) scoping a new idea. Works through a short template section by section:
- **Problem Statement** — the pain or opportunity, not the solution
- **Users** — who it's for and what they're trying to accomplish
- **Goals & Success Signals** — what changes if this works
- **Scope** — what's in and what's explicitly out
- **Open Questions** — unknowns to resolve before building

Produces a ~1-page doc that's ready to hand off as input to a tech spec.

```
/carly-product-req
```

### carly-tech-spec

Guided co-authoring of technical design specifications. Works through a fixed template section by section:
- **Problem Statement** — the actual customer problem (not a user story)
- **Goals & Non-Goals** — scope boundaries
- **Current State** — how things work today
- **Proposed Solution** — architecture, data model, APIs
- **Alternatives Considered** — what else was evaluated and why
- **Risks & Open Questions** — unknowns and dependencies
- **Milestones & Sequencing** — incremental delivery plan

Coaches you through each section with targeted questions, pushes back on vague answers, and optionally tests the finished spec with a fresh reader.

```
/carly-tech-spec
```

## GitHub Action Setup

You can run `carly-code-review` automatically on pull requests using [claude-code-action](https://github.com/anthropics/claude-code-action).

### 1. Add a workflow file

Create `.github/workflows/code-review.yml` in your target repo:

```yaml
name: Code Review
on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      id-token: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          plugin_marketplaces: "https://github.com/carlymr/carlys-claude-skills.git"
          plugins: "carly-tools@carlys-claude-skills"
          claude_args: '--allowedTools "Bash(git *),Bash(gh *),Write"'
          prompt: "/carly-code-review ${{ github.event.pull_request.number }}"
```

### 2. Add your API key

Go to your repo's **Settings > Secrets and variables > Actions** and add `ANTHROPIC_API_KEY`.

### 3. Permissions

The workflow needs these permissions:
- **`contents: read`** — to check out the code
- **`pull-requests: write`** — so the review can post comments on the PR
- **`id-token: write`** — required by claude-code-action for OIDC authentication

The `--allowedTools` flag grants the skill permission to run git/gh commands and write temp files (for posting the review report via `--body-file`).

## Auto-Review on `gh pr create` (Local Hook)

If you'd rather review PRs locally the moment Claude opens them, wire up a `PostToolUse` hook that detects `gh pr create` and nudges the agent to invoke `carly-code-review` on the new PR URL — no GitHub Action required.

### 1. Save the hook script

Save this as `~/.claude/hooks/carly-code-review-on-pr-create.sh` and `chmod +x` it:

```bash
#!/usr/bin/env bash
# PostToolUse hook: when `gh pr create` succeeds, inject a reminder telling
# the main agent to run carly-code-review on the new PR in a fresh subagent.
set -euo pipefail

input=$(cat)

tool_name=$(echo "$input" | jq -r '.tool_name // ""')
if [ "$tool_name" != "Bash" ]; then
  exit 0
fi

command=$(echo "$input" | jq -r '.tool_input.command // ""')
if ! echo "$command" | grep -q "gh pr create"; then
  exit 0
fi

output=$(echo "$input" | jq -r '[.tool_response.stdout // "", .tool_response.output // ""] | join("\n")')

pr_url=$(echo "$output" | grep -oE 'https://github\.com/[^/ ]+/[^/ ]+/pull/[0-9]+' | head -n 1 || true)
if [ -z "$pr_url" ]; then
  exit 0
fi

jq -n --arg url "$pr_url" '{
  hookSpecificOutput: {
    hookEventName: "PostToolUse",
    additionalContext: ("A pull request was just opened at \($url). Invoke the carly-code-review skill now with this URL as the argument so a fresh subagent reviews the PR. Do not wait for the user to ask.")
  }
}'
```

The hook never blocks the tool call — on any non-match or parse failure it silently exits 0. On a successful `gh pr create`, it injects a system reminder that nudges the main agent to invoke the skill on the new PR.

### 2. Register the hook in settings.json

Add this to `~/.claude/settings.json` (or your project's `.claude/settings.json`):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/carly-code-review-on-pr-create.sh"
          }
        ]
      }
    ]
  }
}
```

## Repo Structure

```
.claude-plugin/
  marketplace.json        # Marketplace catalog (defines the carly-tools plugin)
skills/
  carly-code-review/SKILL.md  # Code review orchestrator skill
  carly-product-req/SKILL.md  # Product requirements coauthoring skill
  carly-tech-spec/SKILL.md    # Tech spec coauthoring skill
agents/                   # Specialized sub-agents
```
