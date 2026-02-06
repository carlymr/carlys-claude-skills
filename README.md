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

Also works in CI via `claude -p`:

```bash
claude -p "/carly-code-review 123"
```

## Repo Structure

```
.claude-plugin/
  marketplace.json        # Marketplace catalog (defines the carly-tools plugin)
skills/
  carly-code-review/SKILL.md  # Code review orchestrator skill
agents/                   # Specialized sub-agents
```
