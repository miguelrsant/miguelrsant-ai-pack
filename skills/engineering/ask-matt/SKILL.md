---
name: ask-matt
description: Ask which skill or flow fits your situation. A router over the skills in this repo. Use when the user is unsure which skill to use for their task.
metadata:
  author: Matt Pocock (adapted for OpenCode)
  license: MIT
  version: 1.0.0
---

# Ask Matt

You don't remember every skill, so ask.

A **flow** is a path through the skills. Most paths run along one **main flow**, and two **on-ramps** merge onto it. Everything else is standalone, or a vocabulary layer that runs underneath.

## The main flow: idea → ship

The route most work travels. You have an idea and want it built.

1. **`grill-with-docs`** — sharpen the idea by interview. Start here when you **have a codebase**: it's stateful, retaining what it learns in `CONTEXT.md` and ADRs. (No codebase? Use `grill-me` — see Standalone. Both run the same `grilling` skill underneath; `grill-with-docs` is the one that leaves a paper trail.)
2. **Branch — can you settle every question in conversation?** If a question needs a runnable answer (state, business logic, a UI you have to see), detour through a prototype, bridged by **`handoff`** in both directions (see Crossing sessions):
   - **`handoff`** out, then open a fresh session against that file,
   - **`prototype`** to answer the question with throwaway code,
   - **`handoff`** back what you learned, and reference it from the original idea thread.
3. **Branch — is this a multi-session build?**
   - **For structured spec-anchored development** → **`onp-spec-driven`**. Use when you want machine-auditable specs: each acceptance criterion must have a passing test, and the mechanical gate (exit code) decides "done" — not the agent. It covers the full lifecycle: Especificar → Projetar → Tarefas → Plano (parallel execution with git worktrees) → Executar → Auditar → Aprender. Works best after `grill-with-docs` has sharpened the idea.
   - **For lightweight spec → tickets** → **`to-spec`** (turn the thread into a spec), then **`to-tickets`** to split it into tracer-bullet tickets, each declaring its **blocking edges**. On a local tracker that's one file per ticket under `.scratch/<feature>/issues/`, worked blockers-first by hand; on a real tracker the edges become native blocking links, so any ticket whose blockers are done can be grabbed — kick off **`implement`** per ticket, **clearing context between each one**.
   - **No** → **`implement`** right here, in the same context window.

   Either way, **`implement`** builds each issue by driving **`tdd`** internally — one red-green slice at a time — then closes out by running **`code-review`**, a two-axis review (Standards + Spec) of the diff, before committing. Reach for **`tdd`** on its own when you just want to build a concrete behaviour test-first without a full spec, and **`code-review`** on its own whenever you want to review a branch or PR against a fixed point. Use **`onp-spec-driven`** on its own to audit existing code against a spec, check what requirements lack tests, or generate an execution plan for a feature already specced out.

### Context hygiene

Keep steps 1–3 in **one unbroken context window** — don't compact or clear until after `to-tickets` — so the grilling, spec, and tickets all build on the same thinking. Each `implement` then starts fresh, working from the ticket.

If a session approaches the context limit before `to-tickets`, don't push on degraded — `handoff` and continue in a fresh thread.

## On-ramps

A starting situation that generates work, then merges onto the main flow.

- **Bugs and requests piling up** → **`triage`**. It moves issues through triage roles and produces agent-ready issues, which **`implement`** later picks up.

  Triage is only for issues **you didn't create** — bug reports, incoming feature requests, anything that arrives raw. Tickets that `to-tickets` produced are already agent-ready, so **don't triage them**.

- **Something's broken** → **`diagnosing-bugs`**. For the hard ones: the bug that resists a first glance, the intermittent flake, the regression that crept in between two known-good states. It refuses to theorise until it has a **tight feedback loop** — one command that already goes red on *this* bug — then fixes with a regression test. Its post-mortem hands off to **`improve-codebase-architecture`** when the real finding is that there's no good seam to lock the bug down.

- **A huge, foggy effort — a greenfield project or a huge feature build, too big for one session** → **`wayfinder`**, the most cognitively demanding flow here. When the way from here to the destination isn't visible yet, it charts a **shared map** of **decision tickets** on the issue tracker and resolves them one at a time — producing **decisions, not deliverables** — until the fog is pushed back and the way is clear.

  When the map clears, merge onto the main flow at **`to-spec`**, which collapses the map's linked decisions into a buildable plan, then `to-tickets` and `implement` as usual.

## Codebase health

Not feature work — upkeep.

- **`improve-codebase-architecture`** — run whenever you have a spare moment to keep the codebase good for agents to operate in. It surfaces **deepening opportunities**; picking one _generates an idea_ you can take into the main flow at `grill-with-docs`.
- **`codebase-design`** — the bench you design the chosen module on, using deep-module vocabulary.

## Vocabulary underneath

Two model-invoked references that run *beneath* the other skills — each the single source of truth for its vocabulary.

- **`domain-modeling`** — sharpen the project's *domain* language: challenge a fuzzy term, resolve an overloaded word, record a hard-to-reverse decision as an ADR.
- **`codebase-design`** — the deep-module vocabulary (module, interface, depth, seam, adapter, leverage, locality) for designing a module's *shape*.

## Crossing sessions

- **`handoff`** — when a thread is full or you need to branch off (e.g. into a `prototype` session), this compacts the conversation into a markdown file. Open a new session and reference that file to carry the context across.
- **`todowrite`** (OpenCode built-in) — stay in the **same conversation**, tracking progress with structured task lists. Use at **intentional breaks between phases**. `handoff` forks; `todowrite` continues.

## Standalone

Off the main flow entirely.

- **`grill-me`** — the same relentless interview as `grill-with-docs`, but for when you have **no codebase**. Stateless: it saves nothing locally, builds no `CONTEXT.md`.
- **`prototype`** — a small, throwaway program that answers one design question.
- **`research`** — delegate reading legwork to a **background agent**: it investigates a question against **primary sources**, then leaves a cited Markdown file in the repo.
- **`teach`** — learn a concept over multiple sessions.
- **`writing-great-skills`** — reference for writing and editing skills well.

## Precondition

**`setup-matt-pocock-skills`** — run before your first engineering flow to configure the issue tracker, triage labels, and doc layout the other skills assume.