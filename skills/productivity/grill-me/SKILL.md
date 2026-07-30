---
name: grill-me
description: A relentless interview to sharpen a plan or design, without saving any documents. Use when the user wants to stress-test a plan or idea outside of a codebase context.
metadata:
  author: Matt Pocock (adapted for OpenCode)
  license: MIT
  version: 1.0.0
---

Run a grilling session.

## When to Use

- You have an idea but no codebase yet
- You want to think through a plan before starting
- You need to align on a design before writing code
- You want to stress-test requirements without committing to docs

## What It Does

1. Interviews you relentlessly about every aspect of your plan
2. Walks down each branch of the decision tree
3. Resolves dependencies between decisions one-by-one
4. For each question, provides a recommended answer

## Rules

- Ask questions **one at a time** — waiting for feedback on each question before continuing
- If a *fact* can be found by exploring the environment (filesystem, tools, etc.), look it up rather than asking
- The *decisions* are the user's — put each one to them and wait for their answer
- Do NOT act on anything until the user confirms shared understanding is reached
- Do NOT save any documents, create files, or modify the workspace

## Completion Criterion

Done when the user says they have reached a shared understanding. Do not proceed to implementation.
