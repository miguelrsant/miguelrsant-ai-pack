---
name: grill-me
description: Interviews the user relentlessly about a plan until shared understanding is reached, before any code is written. Use when the task scope is ambiguous, requirements are unclear, or before starting any non-trivial feature.
metadata:
  origin: community
  source: https://github.com/anthropics/skills
---

# Grill Me

Before writing a single line of code, interview the user to pressure-test the plan.

## When to Activate

- The user says "build X", "implement Y", or "add feature Z" without clear requirements
- Task involves more than 3 files or touches multiple systems
- User skips planning and jumps to "write code"
- Any feature that could have multiple valid interpretations
- The task description is one sentence or vague

## Workflow

### Step 1: Detect Lack of Clarity

If the request is vague, incomplete, or ambiguous, do NOT start coding. Instead, say:

> "I'd like to grill this plan before we start. A few questions to make sure we build the right thing."

### Step 2: Ask 3-5 Targeted Questions

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

## Quality Criteria

- No code written before shared understanding is confirmed
- Questions are specific, not generic
- Scope boundaries are explicitly captured
- Trade-offs are surfaced before implementation
- User confirms plan before execution begins
