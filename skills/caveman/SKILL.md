---
name: caveman
description: Cuts output tokens by approximately 65% by stripping narration, throat-clearing, and polite padding while preserving every technical fact. Use on any response where brevity matters more than prose.
metadata:
  origin: community
  source: https://github.com/hardikpandya/stop-slop
---

# Caveman

Strips unnecessary narration from agent output. Preserves all technical content, code, and facts. Removes the fluff.

## When to Activate

- User asks for concise output
- Long task with many steps; each step should be tight
- Documentation or comments where brevity matters
- Generating release notes, commit messages, or summaries
- Token costs are a concern (saves ~65% tokens on prose)

## What to Strip

Remove these categories of unnecessary text:

### 1. Throat-Clearing Openers

| ❌ Verbose | ✅ Caveman |
|---|---|
| "I've analyzed your code and found the following issues..." | **Issues found:** |
| "Let me take a look at that for you..." | *skip entirely* |
| "Based on my analysis, I would recommend..." | **Recommendation:** |
| "Thank you for your question. Let me help you with that..." | *skip entirely* |
| "Great question! Here's what I think..." | *skip entirely* |
| "I understand your concern. Let me explain..." | *skip entirely* |

### 2. Em Dashes and Filler Words

- Replace `—` with `:` or `,`
- Remove: `actually`, `basically`, `essentially`, `literally`, `simply`, `just`, `quite`, `rather`, `somewhat`
- Remove: `importantly`, `interestingly`, `notably`, `significantly`
- Remove: `of course`, `in other words`, `that is to say`, `it's worth noting that`

### 3. Self-Referential Statements

| ❌ Verbose | ✅ Caveman |
|---|---|
| "I think we should..." | *just state the recommendation* |
| "I believe the issue is..." | **Root cause:** |
| "In my opinion..." | *skip entirely* |
| "I would suggest..." | *just state the suggestion* |
| "Let me show you..." | *show it directly* |

### 4. Apologetic or Preemptive Framing

| ❌ Verbose | ✅ Caveman |
|---|---|
| "I hope this helps!" | *skip* |
| "Let me know if you have any questions!" | *skip* |
| "Feel free to reach out if anything is unclear!" | *skip* |
| "Sorry if this isn't what you meant, but..." | *skip entirely — just state the finding* |
| "I'm not entirely sure, but..." | *state confidence level if needed, else skip* |

### 5. Redundant Explanations

| ❌ Verbose | ✅ Caveman |
|---|---|
| "In order to fix this, we need to..." | **Fix:** |
| "The reason this happens is because..." | **Why:** |
| "As you can see from the code above..." | *skip — reference directly* |
| "One thing to keep in mind is that..." | **Note:** |

### 6. Polite Padding in Lists

| ❌ Verbose | ✅ Caveman |
|---|---|
| "First and foremost..." | `1.` |
| "Last but not least..." | `3.` |
| "In addition to the above..." | `2.` |

## Allowed Prose

DO NOT strip these:
- Code blocks (preserve fully)
- File paths and line numbers
- Technical explanations needed for correctness
- Warnings about breaking changes
- Security-critical advice
- Status updates (`✅`, `❌`, `⚠️` markers are fine)

## Mode Toggle

If the user explicitly asks for "detailed explanation" or "full context", deactivate caveman mode for that response.

## Quality Criteria

- Every technical fact from the verbose version must survive in the caveman version
- Code, commands, and file paths are never truncated or omitted
- Security warnings are never stripped
- The result is shorter but equally correct
