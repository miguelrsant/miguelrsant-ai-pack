---
name: skill-optimizer
description: Mines session history for skill-worthy workflows, personalizes installed skills to user habits, and creates or improves skills based on repeated patterns.
metadata:
  origin: community
  source: https://github.com/hqhq1025/skill-optimizer
---

# Skill Optimizer

A three-part toolkit: mine, personalize, and create skills from real usage patterns.

## When to Activate

- User repeats the same multi-step workflow more than twice
- User asks "can you do this faster?" or "this seems repetitive"
- User wants to create a new skill from an existing workflow
- Reviewing or improving existing skills
- User asks to audit which skills are most/least used

## Part 1: Mine Your Workflows

Analyze the current session and available context to identify **skill-worthy patterns**:

### What to Look For

| Pattern | Example | Skill Idea |
|---|---|---|
| Repeated 3+ step sequence | Create branch → make changes → commit → push | `git-flow` |
| Complex checklist you repeat | Start project → install deps → configure → deploy | `project-init` |
| Debug ritual | Check logs → identify error → search fix → apply | `debug-flow` |
| Review pattern | Security check → lint → test → perf review | `review-flow` |

### When a Pattern Is Skill-Worthy

It passes if **3 out of 5** are true:
1. Same steps repeated 2+ times
2. Steps are always in the same order
3. Takes more than 30 seconds to explain each time
4. Involves 3+ distinct tools or commands
5. Another team member would benefit from automation

## Part 2: Personalize Existing Skills

Review locally installed skills and suggest improvements:

### Personalization Checklist

- [ ] Does the skill match your actual tech stack?
- [ ] Are the example commands using your preferred tools?
- [ ] Do the version numbers match your environment?
- [ ] Is the activation logic specific enough?
- [ ] Could the skill be merged with another similar one?

### Improvement Template

```markdown
## Suggested Improvement: [skill-name]

**Current gap:** [what it doesn't handle]
**Proposed change:** [specific edit]
**Rationale:** [why it matters for your workflow]
```

## Part 3: Generate New Skill Skeletons

When mining finds a clear pattern, generate a skill skeleton:

```markdown
---
name: your-skill-name
description: [one-line what it does]
metadata:
  origin: session-mined
---

# [Skill Name]

## When to Activate

[Specific triggers based on the mined pattern]

## Workflow

1. [Step 1 from the observed pattern]
2. [Step 2]
3. [Step 3]

## Quality Criteria

- [What must be true for this workflow to succeed]
```

## Output Format

When mining is requested, produce:

```markdown
# Skill Mining Report

## Patterns Found

1. **Pattern: [name]** (observed N times)
   - Steps: [step list]
   - Recommended: Create/Fix skill `[name]`

## Skills to Create

- [skill-name] — [brief description]

## Skills to Improve

- [skill-name] — [suggested improvement]

## Quick Wins

- [Smallest change with biggest impact]
```

## Quality Criteria

- Patterns are based on real observed behavior, not guesses
- Skill suggestions are specific, not generic
- Improvements are actionable (including exact file/line references)
- New skills follow the project's SKILL.md format conventions
