---
name: code-review
description: Review the changes since a fixed point (commit, branch, tag, or merge-base) along two axes — Standards (does the code follow documented coding standards?) and Spec (does the code match what the originating issue/PRD asked for?). Runs both reviews in parallel and reports them side by side. Use when the user wants to review a branch, a PR, work-in-progress changes, or asks to "review since X".
metadata:
  author: Matt Pocock (adapted for OpenCode)
  license: MIT
  version: 1.0.0
---

Two-axis review of the diff between `HEAD` and a fixed point the user supplies:

- **Standards** — does the code conform to this repo's documented coding standards?
- **Spec** — does the code faithfully implement the originating issue / PRD / spec?

Both axes are conducted as separate analytical passes so they don't pollute each other, then findings are aggregated.

## Process

### 1. Pin the fixed point

Whatever the user said is the fixed point — a commit SHA, branch name, tag, `main`, `HEAD~5`, etc. If they didn't specify one, ask for it.

Capture the diff command once: `git diff <fixed-point>...HEAD` (three-dot, so the comparison is against the merge-base). Also note the list of commits via `git log <fixed-point>..HEAD --oneline`.

Before going further, confirm the fixed point resolves (`git rev-parse <fixed-point>`) and the diff is non-empty.

### 2. Identify the spec source

Look for the originating spec, in this order:

1. Issue references in the commit messages (`#123`, `Closes #45`, etc.)
2. A path the user passed as an argument
3. A PRD/spec file under `docs/`, `specs/`, or `.scratch/` matching the branch name or feature
4. If nothing is found, ask the user where the spec is

### 3. Identify the standards sources

Anything in the repo that documents how code should be written, such as `CODING_STANDARDS.md`, `CONTRIBUTING.md`, or the `rules/` directory.

On top of whatever the repo documents, the Standards axis always carries the **smell baseline** below:

- **Mysterious Name** — a function, variable, or type whose name doesn't reveal what it does. → rename it.
- **Duplicated Code** — same logic shape appears in more than one place. → extract the shared shape.
- **Feature Envy** — a method that reaches into another object's data more than its own. → move the method.
- **Data Clumps** — same fields/params keep travelling together. → bundle into one type.
- **Primitive Obsession** — a primitive standing in for a domain concept. → give it its own type.
- **Repeated Switches** — same switch/if-cascade on the same type recurs. → replace with polymorphism.
- **Shotgun Surgery** — one logical change forces scattered edits. → gather into one module.
- **Divergent Change** — one file edited for several unrelated reasons. → split the file.
- **Speculative Generality** — abstraction added for needs the spec doesn't have. → delete it.
- **Message Chains** — long `a.b().c().d()` navigation. → hide behind one method.
- **Middle Man** — a class that mostly delegates onward. → cut it, call the real target.
- **Refused Bequest** — a subclass that overrides most of what it inherits. → use composition.

### 4. Run both reviews

**Standards review**: Examine the diff against documented standards + smell baseline. Report per file/hunk:
- Every place the diff violates a documented standard
- Any baseline smell spotted
- Distinguish hard violations from judgement calls
- Skip anything tooling already enforces

**Spec review**: Compare the diff against the originating spec. Report:
- Requirements the spec asked for that are missing or partial
- Behaviour in the diff that wasn't asked for (scope creep)
- Requirements that look implemented but the implementation looks wrong

### 5. Aggregate

Present both reports under `## Standards` and `## Spec` headings. Do **not** merge or rerank findings — the two axes are deliberately separate.

End with a one-line summary: total findings per axis, and the worst issue *within each axis*.

## Why Two Axes

A change can pass one axis and fail the other:

- Code that follows every standard but implements the wrong thing → **Standards pass, Spec fail.**
- Code that does exactly what the issue asked but breaks conventions → **Spec pass, Standards fail.**

Reporting them separately stops one axis from masking the other.
