# AI Pack — Architecture

## Language

- Always communicate with the user in **Portuguese (Brazilian)**
- Write code comments in English for international compatibility
- Documentation and commit messages should be in English

## Core Architecture

This pack follows a **Pure Orchestration** model:

```
User Request → Orchestrator (delegates) → Specialist Agent (executes) → Result
```

### Orchestrator
- **NEVER writes code, edits files, or runs bash**
- Has only Read, Grep, Glob, Task, skill, and Todowrite tools
- Knows ALL 28 agents and 91 skills in the repository
- Routes requests to the correct specialist agent
- If no agent exists for a task, STOPS and asks the user

### Agents
- Each agent is **specialized** in a specific domain
- Each agent has **skills assigned** that it can load for guidance
- Agents either have Write/Edit/Bash (executors) or are READ-ONLY (reviewers)

---

## Available Agents (28)

### Planning & Architecture
| Agent | Skills | Executes? |
|---|---|---|
| `planner` | search-first, backend-patterns, frontend-patterns, database-migrations, coding-standards | ❌ Read-only |
| `architect` | backend-patterns, frontend-patterns, django-patterns, python-patterns, api-design, postgres-patterns, adr, codebase-design | ❌ Read-only |

### Backend Implementation
| Agent | Skills | Executes? |
|---|---|---|
| `backend` | django, drf, django-patterns, django-security, django-drf-production, django-tdd, django-verification, python-patterns, database-migrations, api-contracts-openapi, api-design, django-celery | ✅ Yes |

### Code Review
| Agent | Skills | Executes? |
|---|---|---|
| `code-reviewer` | security-review, coding-standards, backend-patterns, frontend-patterns, python-patterns | ❌ Read-only |
| `python-reviewer` | python-patterns, python-testing, django-security, django-verification, error-handling | ❌ Read-only |
| `typescript-reviewer` | frontend-patterns, backend-patterns, react-vite-tailwind-integration, coding-standards | ❌ Read-only |
| `database-reviewer` | postgresql, postgres-patterns, database-migrations, mysql-patterns, redis-patterns, django | ❌ Read-only |
| `security-reviewer` | security-review, django-security, security-scan | ❌ Read-only |
| `api-reviewer` | api-contracts-openapi, api-design, django-rest-framework, contract-first | ❌ Read-only |

### Testing & QA
| Agent | Skills | Executes? |
|---|---|---|
| `tdd` | tdd-workflow, django-tdd, python-testing, testing, e2e-testing, ai-regression-testing | ✅ Yes |
| `e2e` | e2e-testing, testing | ✅ Yes |
| `qa-agent` | testing, python-testing, e2e-testing, tdd-workflow, ai-regression-testing, error-handling | ✅ Yes |

### Build Resolution
| Agent | Skills | Executes? |
|---|---|---|
| `build-error-resolver` | (generic) | ✅ Yes |
| `react-build-resolver` | react-patterns, vite-patterns, react-performance, ui-styling | ✅ Yes |
| `django-build-resolver` | django, drf, django-celery, database-migrations, docker | ✅ Yes |

### Framework Review
| Agent | Skills | Executes? |
|---|---|---|
| `react-reviewer` | react-patterns, react-performance, react-testing, frontend-a11y, frontend-patterns, ui-styling | ❌ Read-only |
| `fastapi-reviewer` | fastapi-patterns, python-patterns, api-design, postgres-patterns | ❌ Read-only |
| `django-reviewer` | django, drf, django-patterns, django-security, django-tdd, django-verification | ❌ Read-only |

### Performance & Quality
| Agent | Skills | Executes? |
|---|---|---|
| `performance-optimizer` | react-performance, frontend-patterns, postgres-patterns, python-patterns, backend-patterns | ✅ Yes |
| `silent-failure-hunter` | error-handling, python-patterns, coding-standards | ❌ Read-only |
| `code-simplifier` | coding-standards, python-patterns, error-handling | ✅ Yes |
| `code-explorer` | **DEFAULT** — typescript-patterns, frontend-patterns, backend-patterns, coding-standards, error-handling, readme-architecture-docs | ❌ Read-only (has Bash) |
| `spec-miner` | (self-contained) | ⚠️ Limited write |

### DevOps & Infrastructure
| Agent | Skills | Executes? |
|---|---|---|
| `devops-specialist` | docker, docker-patterns, ci-cd, deployment-patterns, production-readiness, production-audit, github-actions, security-review | ✅ Yes |
| `production-reviewer` | production-readiness, deployment-patterns, docker, docker-patterns, ci-cd, github-actions, production-audit | ❌ Read-only |

### Design
| Agent | Skills | Executes? |
|---|---|---|
| `ui-ux-designer` | ui-ux-pro-max (PRIMARY), ui-styling, frontend-patterns, react-patterns, frontend-a11y, react-performance, react-vite-tailwind-integration, frontend-design-direction, design-system, design, banner-design, brand, slides, framer-motion | ✅ Yes |

### Research & Search
| Agent | Skills | Executes? |
|---|---|---|
| `search-researcher` | deep-research (PRIMARY), research, search-first, claude-video | ✅ Yes (web access) |

### Documentation
| Agent | Skills | Executes? |
|---|---|---|
| `documentation` | readme-architecture-docs, api-contracts-openapi | ✅ Yes |

---

## Skills Reference (91 skills)

### How Skills Work
Skills are instructions loaded via the `skill` tool. Each agent should load its assigned skills before working.

### Skill Categories

| Category | Skills |
|---|---|
| **Design & Frontend UI** (10) | ui-ux-pro-max, ui-styling, frontend-design-direction, frontend-patterns, frontend-a11y, frontend-slides, design, design-system, brand, banner-design, slides |
| **React & Framework** (8) | react-patterns, react-performance, react-testing, react-native-patterns, react-vite-tailwind-integration, vite-patterns, nextjs-turbopack, bun-runtime, framer-motion |
| **Django & Backend** (11) | django, django-rest-framework, django-patterns, django-security, django-tdd, django-verification, django-drf-production, django-celery, fastapi-patterns, backend-patterns, python-patterns, pytorch-patterns |
| **API & Contracts** (3) | api-design, api-contracts-openapi, contract-first |
| **Database** (6) | postgresql, postgres-patterns, mysql-patterns, database-migrations, prisma-patterns, redis-patterns |
| **Testing** (6) | tdd-workflow, testing, python-testing, e2e-testing, ai-regression-testing, react-testing |
| **Security** (3) | security-review, security-scan, django-security |
| **DevOps** (7) | docker, docker-patterns, ci-cd, github-actions, deployment-patterns, production-readiness, production-audit |
| **Architecture & Quality** (5) | architecture-decision-records, coding-standards, codebase-design, error-handling, mcp-server-patterns |
| **Research** (4) | deep-research, research, search-first, claude-video |
| **Planning & Process** (6) | ask-matt, grill-with-docs, grill-me, grilling, to-spec, to-tickets, implement, wayfinder, triage, diagnosing-bugs |
| **Code Improvement** (4) | improve-codebase-architecture, codebase-design, domain-modeling, prototype, resolving-merge-conflicts |
| **Productivity** (5) | handoff, teach, writing-great-skills, caveman |
| **Tooling** (9) | agent-creator, skill-creator, skill-optimizer, git-commit, git-workflow, git-guardrails-claude-code, setup-pre-commit, mcp-setup, readme-architecture-docs |

---

## Code Standards

- Follow PEP 8 for Python code
- Follow TypeScript best practices
- Use type hints in Python
- Write tests with pytest (Python) and Vitest (TypeScript)
- All agents must respect their assigned skill sets
- Orchestrator NEVER implements code — delegates everything
