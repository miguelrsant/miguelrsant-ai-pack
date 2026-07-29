---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea until shared understanding is reached. Optionally creates ADRs and glossary as you go (grill-with-docs mode). Use when the task scope is ambiguous, requirements are unclear, or before starting any non-trivial feature.
---

# Grilling

Interview the user relentlessly before writing a single line of code — pressure-test the plan, surface hidden assumptions, and make sure everyone builds the right thing.

## When to Activate

- The user says "build X", "implement Y", or "add feature Z" without clear requirements
- Task involves more than 3 files or touches multiple systems
- User skips planning and jumps to "write code"
- Any feature that could have multiple valid interpretations
- The task description is one sentence or vague
- The user explicitly says "grill me", "grill this", or any similar trigger phrase

## Instructions

Interview me relentlessly about every aspect of this until we reach a shared understanding. Walk down each branch of the decision tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time, waiting for feedback on each question before continuing. Asking multiple questions at once is bewildering.

If a *fact* can be found by exploring the environment (filesystem, tools, etc.), look it up rather than asking me. The *decisions*, though, are mine — put each one to me and wait for my answer.

Do not act on it until I confirm we have reached a shared understanding.

## Workflow

### Step 1: Detect Lack of Clarity

If the request is vague, incomplete, or ambiguous, do NOT start coding. Instead, say:

> "I'd like to grill this plan before we start. A few questions to make sure we build the right thing."

### Step 2: Ask Targeted Questions

Customize questions to the domain. General categories:

**Goal questions:**
- "What is the primary user goal this should solve?"
- "What does success look like — how will we know it works?"

**Scope questions:**
- "What is explicitly in scope? What is out of scope?"
- "Are there any constraints (time, budget, tech stack) I should know about?"

**Edge case questions:**
- "What happens if the input is invalid, the network fails, or the user is unauthorized?"
- "What's the error scenario we should handle first?"

**Trade-off questions:**
- "Do you prefer simpler code now or a more extensible structure?"
- "Is performance more important than readability here?"

### Step 3: Summarize Shared Understanding

After the user answers, restate your understanding:

> "So to confirm: we're building [X] that does [Y] for [Z]. It should handle [edge case]. We'll prioritize [trade-off]. Correct?"

### Step 4: Proceed Only on Confirmation

Wait for explicit confirmation ("yes", "correct", "go ahead") before coding. If the user says "just do it" without answering, pick reasonable defaults and note them.

## Example Interaction

```
User: "Add a dark mode toggle to the settings page."
Agent: "I'd like to grill this plan first.
1. Should the toggle persist across sessions (localStorage, cookie, server-side)?
2. Does dark mode apply to all pages or just settings?
3. Do you have a design spec, or should I use system-contrast ratios?"
```

## Optional: Grill with Docs

For high-stakes or complex designs, run grilling in **grill-with-docs mode** — a relentless interview that also captures the output as structured documentation.

When activated, the agent:
1. Loads the `/domain-modeling` skill alongside grilling
2. Creates **ADR entries** (Architecture Decision Records) under `docs/adr/` as each decision crystalises
3. Maintains a **glossary** of domain terms in `CONTEXT.md` (or `docs/glossary.md`)
4. Links decisions to glossary terms so future developers understand *why* the codebase is shaped the way it is

This mode is ideal when:
- The design has long-lived architectural consequences
- Multiple developers will work on the same area
- The domain vocabulary is new or evolving
- You want a permanent record of why decisions were made

## Quality Criteria

- No code written before shared understanding is confirmed
- Questions are asked one at a time (never batched)
- Questions are specific, not generic
- Scope boundaries are explicitly captured
- Trade-offs are surfaced before implementation
- User confirms plan before execution begins
- Facts are looked up in the environment, not asked of the user
- In grill-with-docs mode: ADRs and glossary are created as decisions are made
