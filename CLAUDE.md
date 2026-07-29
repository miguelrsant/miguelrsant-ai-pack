# AI Pack — Architecture

## Language

- Communicate with user in **Portuguese (Brazilian)**
- Code comments in English for international compatibility
- Documentation and commit messages in English

## Core Architecture

```
User Request → Orchestrator (discovers + routes) → Specialist Agent (executes) → Result
```

### Orchestrator
- **NEVER writes code, edits files, or runs bash**
- Uses only Read, Grep, Glob, Task, skill, Todowrite
- Discovers agents dynamically via `glob agents/*.md` + frontmatter parsing
- Routes via scoring algorithm based on agent metadata
- If no agent exists, STOPS and asks the user
- Full orchestration prompt: `agents/orchestrator.md`

### Agents
- Each agent has standardized YAML frontmatter with:
  - `type`: executor (writes code) or reviewer (read-only)
  - `capabilities`: what the agent can do
  - `technologies`: languages/frameworks it supports
  - `task_types`: what types of tasks it handles
  - `priority`: routing priority (higher = more specialized)
  - `skills_used`: skills it can load for guidance
- Agents either have Write/Edit/Bash (executors) or are READ-ONLY (reviewers)
- Adding a new agent = creating a new file in `agents/`

## Code Standards

- PEP 8 for Python, type hints preferred
- TypeScript best practices (strict mode)
- Tests with pytest (Python) and Vitest (TypeScript)
- All agents respect their assigned skills
- Orchestrator NEVER implements code — delegates everything
