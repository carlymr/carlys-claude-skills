---
name: carly-tech-spec
description: "Co-author a technical design specification through structured conversation. Guides you through problem statement, goals, current state, proposed solution, alternatives, risks, and milestones — with a fixed template and coaching at each step. Use when writing a tech spec, design doc, technical proposal, or RFC."
allowed-tools: Read, Write, Edit, Glob, Grep, Task
---

# Tech Spec Co-Authoring

Guide the user through writing a technical design spec using a fixed template. Work section by section, asking targeted questions and drafting iteratively.

## Step 1 — Setup

Read `references/template.md` (relative to this skill's directory) to load the template structure.

Ask the user:
1. What is this spec about? (one sentence)
2. Who is the audience? (engineering team, leadership, cross-functional, etc.)

Create a spec file in the working directory named `tech-spec-<short-name>.md` using the template skeleton from `references/template.md`. Fill in the title, author (ask if unknown), date, and set status to "Draft". Strip the guidance comments (lines starting with `>`) — those are only for your reference during authoring.

Tell the user the file has been created and that you'll work through each section together.

## Step 2 — Problem Statement

This is the most important section. Do not rush it.

Ask the user to describe the problem in their own words. Then ask clarifying questions:
- "What's the actual pain users/customers are experiencing?"
- "What happens if this isn't solved?"
- "Why is this the right time to address it?"

Draft the Problem Statement (1-3 paragraphs). Focus on the **problem**, not the solution. Avoid user-story format ("As a user, I want...") — instead describe the real pain and its impact.

Show the draft and iterate. A good problem statement should make someone who knows nothing about the project understand why this work matters.

Update the spec file with the finalized Problem Statement before moving on.

## Step 3 — Remaining sections

Work through each section in template order: Goals & Non-Goals → Current State → Proposed Solution → Alternatives Considered → Risks & Open Questions → Milestones & Sequencing.

For each section:

1. **Ask 3-5 targeted questions** based on what you know so far. Reference earlier sections to stay consistent.
2. **Draft the section** from their answers. Use the guidance in `references/template.md` to calibrate depth and tone.
3. **Show the draft and iterate.** Make surgical edits — don't rewrite whole sections unless asked.
4. **Update the spec file** when the section is finalized.

Section-specific guidance:

- **Goals & Non-Goals**: Push for explicit non-goals. Ask "What might someone assume is in scope that actually isn't?"
- **Current State**: Ask for existing architecture, diagrams, or links. If the user is working in a codebase, offer to explore it with Glob/Grep/Read to understand the current state.
- **Proposed Solution**: Encourage specificity — data models, API contracts, pseudocode. Ask about edge cases. If the solution is vague, push back: "How would another engineer implement this from what's written here?"
- **Alternatives Considered**: Proactively suggest 1-2 alternatives the user may not have considered, based on the problem and constraints. Ask why each was rejected.
- **Risks & Open Questions**: Surface risks the user hasn't mentioned — based on the proposed solution's complexity, dependencies, or unknowns. Ask who needs to answer each open question.
- **Milestones & Sequencing**: Push for incremental delivery. Ask "What's the smallest thing you could ship first to validate the approach?"

## Step 4 — Review & polish

Re-read the full spec file. Check for:
- Does the proposed solution actually solve the problem statement?
- Are there contradictions between sections?
- Is anything redundant or filler?
- Would the target audience understand this without additional context?

Suggest specific edits. Apply them after user approval.

Update the spec's status field to "In Review" when the user is satisfied.

## Step 5 — Reader test (optional)

Offer to test the spec with a fresh perspective.

**If sub-agents are available** (Claude Code): spawn a Task agent with `subagent_type: "general-purpose"`. Pass it only the spec content (not the conversation history). Ask it to:
1. Summarize the proposal in 2-3 sentences
2. Identify any gaps, ambiguities, or assumptions
3. List questions a reviewer would likely ask

Report the findings and fix any gaps by looping back to the relevant section.

**If sub-agents aren't available**: suggest the user paste the spec into a fresh Claude conversation and ask the same questions.

## Tone

- Be direct and collaborative, not performative
- Push back when sections are vague — a spec that glosses over details isn't useful
- Don't pad sections with filler to make them look complete
- Respect the user's time: if a section doesn't apply, skip it (but confirm first)
