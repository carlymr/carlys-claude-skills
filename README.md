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

Comprehensive code review using 6 parallel specialized sub-agents:
- **Correctness** — bugs, logic errors, missing edge cases
- **Security** — vulnerabilities, injection, auth flaws, secrets exposure
- **Performance** — algorithmic complexity, N+1 queries, unnecessary allocations
- **Simplicity** — over-engineering, unnecessary abstractions
- **UX** — confusing APIs, poor error messages, accessibility
- **Codebase Integration** — duplicated functionality, pattern mismatches

The orchestrator assesses project maturity (prototype vs production) to calibrate review rigor, then synthesizes all findings — verifying against actual code, de-duplicating, and dropping minor comments that would resolve when fixing larger issues.

Supports local git diff (no arguments) or GitHub PR review with inline comments.

```
/carly-code-review
/carly-code-review 123
/carly-code-review https://github.com/org/repo/pull/123
```

Also works in CI — see [GitHub Action Setup](#github-action-setup) below.

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

## Repo Structure

```
.claude-plugin/
  marketplace.json        # Marketplace catalog (defines the carly-tools plugin)
skills/
  carly-code-review/SKILL.md  # Code review orchestrator skill
  carly-tech-spec/SKILL.md    # Tech spec coauthoring skill
agents/                   # Specialized sub-agents
```
