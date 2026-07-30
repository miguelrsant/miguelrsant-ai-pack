---
name: grill-with-docs
description: A relentless interview to sharpen a plan or design, which also creates docs (ADRs and glossary) as you go. Use when the user has an idea that needs refining and there's a codebase to document decisions in.
metadata:
  author: Matt Pocock (adapted for OpenCode)
  license: MIT
  version: 1.0.0
---

Run a grilling session, using the domain-modeling skill in parallel.

## Process

1. Load the `grilling` skill to interview the user relentlessly about every aspect of the plan
2. Load the `domain-modeling` skill in parallel to capture decisions as they crystallize
3. As terms are resolved, update `CONTEXT.md` with the shared glossary
4. When hard-to-reverse decisions are made, offer to create an ADR in `docs/adr/`
5. Continue until every branch of the decision tree is resolved

## Key Differences from grill-me

| Aspect | grill-with-docs | grill-me |
|--------|----------------|----------|
| Use when | You have a codebase | No codebase |
| Stateful? | Yes — saves to CONTEXT.md + ADRs | No — stateless |
| Output | Glossary + ADRs + shared understanding | Just shared understanding |

## Side Effects

- **Naming a concept not in `CONTEXT.md`?** Add the term. Create the file lazily if it doesn't exist.
- **Sharpening a fuzzy term?** Update `CONTEXT.md` right there.
- **User rejects a direction with a load-bearing reason?** Offer an ADR.
