# Tech Spec Template

Use this template to scaffold the spec document. Replace bracketed placeholders with actual content. Delete guidance comments (lines starting with `>`) before finalizing.

---

# [Title]

**Author:** [name]
**Date:** [date]
**Status:** Draft | In Review | Approved
**Reviewers:** [names]

## Problem Statement

> This is the most important section. Describe a pain that real users or customers are experiencing in their lives or work today — something that's broken, frustrating, costly, or impossible for them right now. The test: if no one ever built this product, would someone still be hurting? If yes, you have a problem worth solving.
>
> Answer these questions:
> - Who is feeling this pain, and how do you know? (What have they said or done?)
> - What are they trying to accomplish, and what's getting in their way?
> - What's the cost to them of the status quo — time, money, missed outcomes, frustration?
> - What do they do today instead? (Workarounds, competitors, giving up.)
> - Why now? What's changed for them that makes this more pressing?
>
> Keep the focus on the user or customer throughout. Business-side outcomes you want (revenue, growth, launches) are goals, not problems — they belong in the Goals section. If the only "problem" you can name is "we want to monetize" or "we want to grow," the spec is likely premature; the user problem isn't clear enough yet.
>
> Cost/revenue framings deserve extra scrutiny. "Infra costs outpace revenue" is a symptom, not a problem. Ask: is the infra solving a real user problem? If yes, that user problem is the problem statement (and monetization is a solution). If no, the answer is to stop spending the money, not to charge for it.
>
> Keep it to 1-3 paragraphs. Don't name the proposed solution here.

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
