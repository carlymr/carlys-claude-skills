# Carly's Claude Skills

This is a Claude Code plugin marketplace repo. The marketplace defines a single plugin (`carly-tools`) with `strict: false`, so skills and agents live at the repo root.

## Repo structure

- `.claude-plugin/marketplace.json` — marketplace catalog; defines the `carly-tools` plugin and lists which skills it includes
- `skills/<skill-name>/SKILL.md` — skill definitions
- `agents/` — sub-agent definitions (markdown files with YAML frontmatter)

## When adding a new skill

1. Create `skills/<skill-name>/SKILL.md`
2. Add any sub-agents to `agents/`
3. Add the skill path to the `skills` array in `.claude-plugin/marketplace.json`
4. Follow the skill-creator guidelines: keep SKILL.md under 500 lines, use progressive disclosure for reference material, write concise descriptions that include trigger conditions

## Conventions

- Skill and agent names use kebab-case
- Skill descriptions should be comprehensive — they're the primary trigger mechanism
- Sub-agents should be read-only (`tools: Read, Grep, Glob`) unless they need to modify files
- Prefer `model: sonnet` for sub-agents to balance capability and speed
- Keep orchestrator skills lean; delegate domain-specific checklists to sub-agents or reference files

## Current skills

- **carly-code-review** — orchestrator + 6 reviewer sub-agents for comprehensive code review
- **carly-tech-spec** — guided tech spec coauthoring with fixed template and Problem Statement emphasis
