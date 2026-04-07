---
name: carly-product-req
description: "Co-author a lightweight product requirements document through structured conversation. Guides non-technical teammates through problem statement, users, goals, success signals, and open questions — producing a short doc (~1 page) that's ready to hand off for a tech spec. Use when writing product requirements, a PRD, feature brief, or scoping a new product idea."
allowed-tools: Read, Write, Edit, Glob, Grep, Task
---

# Product Requirements Co-Authoring

Guide the user through writing a lightweight product requirements doc. Work section by section, asking targeted questions and drafting iteratively. The goal is mutual understanding — enough clarity for an engineer to pick this up and start a tech spec — not a comprehensive specification.

## Audience awareness

The user is likely non-technical (PM, designer, founder). Avoid jargon. Ask questions in plain language. Coach them toward clarity without being condescending.

## Step 1 — Setup

Read `references/template.md` (relative to this skill's directory) to load the template structure.

Ask the user:
1. What are we building? (one sentence)
2. Who's going to read this? (engineering team, cofounder, investors, etc.)

Create a requirements doc in the working directory named `product-req-<short-name>.md` using the template skeleton from `references/template.md`. Fill in the title, author (ask if unknown), date, and set status to "Draft". Strip the guidance comments (lines starting with `>`) — those are only for your reference during authoring.

Tell the user the file has been created and that you'll work through each section together.

## Step 2 — Problem Statement

This is the most important section. Spend the most time here.

Ask the user to describe the problem or opportunity in their own words. Then ask clarifying questions:
- "What's the pain or gap you're seeing?"
- "If this already sort of exists somewhere, why isn't that good enough?"
- "If this is brand new — what makes you believe people need it?"
- "What happens if you don't build this?"
- "Why now?"

Draft the Problem Statement (1-2 paragraphs). Focus on the **problem or opportunity**, not the solution. If the user jumps to describing a solution, gently redirect: "Let's capture *why* this matters first — we'll get to the *what* next."

Show the draft and iterate. A good problem statement should make someone with no context understand why this work matters.

Update the doc with the finalized Problem Statement before moving on.

## Step 3 — Users

Ask:
- "Who specifically is this for? A type of user, a role, a segment?"
- "What are they trying to accomplish?"
- "How do they handle this today — or do they not at all?"
- If they don't do it today: "What's changed that makes this worth building now?"
- If they do something today: "What's frustrating or limiting about their current approach?"

Keep this section short — a paragraph or a few bullets. The goal is to ground the rest of the doc in a real person with a real need.

Update the doc before moving on.

## Step 4 — Goals & Non-Goals

Ask:
- "If this goes well, what's true that isn't true today?"
- "What might someone assume is in scope that actually isn't — at least not right now?"

Push for explicit non-goals. Early-stage teams often try to boil the ocean. Non-goals are a gift to the engineer who picks this up.

Draft goals as concrete outcomes, not tasks. "Users can do X" not "Build X feature."

Update the doc before moving on.

## Step 5 — How We'll Know It Worked

Ask:
- "If you checked in on this a month after launch, what would tell you it's working?"
- "What's the simplest signal — even anecdotal — that this was the right call?"

Keep this to 1-2 signals. Don't push for a metrics framework — a lightweight indicator is fine. "Users stop asking for X in support" or "3 out of 5 pilot customers adopt it" are perfectly good.

Update the doc before moving on.

## Step 6 — Open Questions

Ask:
- "What are you still unsure about?"
- "What would you want to validate before investing engineering time?"
- "Is there anything you'd want a designer or engineer to weigh in on before you finalize this?"

If the user says "nothing" — push gently. There are almost always open questions at this stage. Surface 1-2 based on what you've heard in earlier sections.

Update the doc.

## Step 7 — Review

Re-read the full doc. Check for:
- Does the problem statement still hold up given everything else?
- Are the goals actually aligned with the problem?
- Is anything contradictory or redundant?
- Would someone picking this up for the first time understand what to build and why?

Suggest specific edits. Apply them after user approval.

Update the status to "In Review" when the user is satisfied.

Mention that this doc is ready to hand off — an engineer can use it as the starting point for a tech spec.

## Tone

- Warm but direct — this is a collaborative conversation, not a form to fill out
- Ask one or two questions at a time, not a wall of five
- If something is vague, push back kindly: "Can you help me get more specific here?"
- Don't pad sections to make them look complete — short and clear beats long and thorough
- If a section genuinely doesn't apply, skip it (but confirm first)
- Celebrate when something clicks — "That's a really clear way to put it" goes a long way with non-technical collaborators
