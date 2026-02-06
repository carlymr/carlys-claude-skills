# Tech Spec Template

Use this template to scaffold the spec document. Replace bracketed placeholders with actual content. Delete guidance comments (lines starting with `>`) before finalizing.

---

# [Title]

**Author:** [name]
**Date:** [date]
**Status:** Draft | In Review | Approved
**Reviewers:** [names]

## Problem Statement

> This is the most important section. Describe the actual customer/user problem — not a feature request, not a user story, not a solution. Focus on the pain: what's broken, what's frustrating, what's impossible today. A good problem statement makes the reader understand *why* this work matters before they see *what* you're proposing.
>
> Answer these questions:
> - What problem are real users/customers hitting?
> - What's the impact — how bad is it, and for how many people?
> - What happens if we don't solve it?
> - Why now? What's changed that makes this urgent or timely?
>
> Keep it to 1-3 paragraphs. If you can't state the problem clearly, the spec isn't ready.

[Problem statement here]

## Goals & Non-Goals

> Goals define what success looks like for this spec. Non-goals are things that are explicitly out of scope — they prevent scope creep and set expectations. A non-goal isn't "we'll never do this," it's "we won't do this *in this effort*."

**Goals:**
- [Goal 1]
- [Goal 2]

**Non-Goals:**
- [Non-goal 1 — brief reason why]
- [Non-goal 2 — brief reason why]

## Current State

> Describe how things work today. What's the existing system/process? Where does it fall short? Include architecture diagrams, data flows, or screenshots if they help. The reader needs to understand the baseline before they can evaluate your proposal.

[Current state description]

## Proposed Solution

> The core of the spec. Describe your technical approach in enough detail that another engineer could implement it. Include:
> - High-level architecture or system design
> - Data model changes (new tables, schema changes, migrations)
> - API changes (new endpoints, modified contracts)
> - Key algorithms or logic
> - How edge cases are handled
>
> Use diagrams, pseudocode, or code snippets where they clarify. Don't write production code here — focus on communicating the design.

[Proposed solution]

## Alternatives Considered

> Show your work. What other approaches did you evaluate? Why were they rejected? This builds confidence that the proposed solution is well-considered, and helps future readers understand the tradeoffs.
>
> For each alternative: 1-2 sentences on what it is, then why it was rejected.

### [Alternative 1]
[Description and why rejected]

### [Alternative 2]
[Description and why rejected]

## Risks & Open Questions

> What could go wrong? What don't you know yet? Be honest — surfacing risks early is better than discovering them during implementation.
>
> **Risks**: things that could cause the project to fail or need significant rework.
> **Open questions**: decisions that haven't been made yet and need input.

**Risks:**
- [Risk 1 — mitigation strategy]
- [Risk 2 — mitigation strategy]

**Open Questions:**
- [Question 1 — who needs to answer this]
- [Question 2 — when does this need to be resolved]

## Milestones & Sequencing

> How will this be built incrementally? Break the work into phases that each deliver value. Avoid a single "big bang" milestone. Consider: what's the smallest thing you could ship first to validate the approach?

| Milestone | Scope | Target |
|-----------|-------|--------|
| [Phase 1] | [What's included] | [Timeframe or dependency] |
| [Phase 2] | [What's included] | [Timeframe or dependency] |
