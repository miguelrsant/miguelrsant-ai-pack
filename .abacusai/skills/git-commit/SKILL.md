---
name: git-commit
description: Generate professional Git commit messages following the Conventional Commits specification.
metadata:
  author: Miguel Angelo
  license: MIT
  version: 1.0.0
---

# Git Commit

## Purpose

Generate professional Git commit messages following the Conventional Commits specification.

## Instructions

When asked to generate a commit message:

1. Analyze the staged changes or the provided diff.
2. Determine the primary purpose of the changes.
3. Select the most appropriate Conventional Commit type.
4. Detect whether a scope should be used.
5. Detect breaking changes when applicable.
6. If multiple unrelated changes are detected, recommend splitting them into separate commits.
7. Generate the final commit message.

## Conventional Commit Format

<type>(<scope>): <description>

The scope is optional and should only be included when it clearly identifies the affected part of the project.

Examples:

feat(auth): add JWT authentication

fix(api): prevent duplicate user registration

refactor(users): simplify repository layer

docs(readme): update installation guide

chore(deps): upgrade Flask dependencies

## Commit Types

- feat: A new feature.
- fix: A bug fix.
- docs: Documentation changes only.
- style: Formatting changes that do not affect code behavior.
- refactor: Code restructuring without changing behavior.
- perf: Performance improvements.
- test: Adding or updating tests.
- build: Build system or dependency changes.
- ci: Continuous Integration or deployment configuration.
- chore: Maintenance tasks.
- revert: Revert a previous commit.

## Scope Guidelines

Use scopes only when they improve clarity.

Common examples:

- auth
- api
- backend
- frontend
- users
- database
- migrations
- docker
- config
- docs
- ui
- cli
- tests
- deps

Avoid generic scopes such as "project" or "misc".

## Breaking Changes

If the change introduces a breaking API change, use one of these formats:

feat(api)!: remove legacy authentication endpoint

or

BREAKING CHANGE: Authentication tokens generated before v2 are no longer valid.

## Rules

- Use imperative mood.
- Keep the subject under 72 characters.
- Do not end the subject with a period.
- Use lowercase commit types.
- Keep the message concise.
- Describe the logical change, not the implementation.
- Prefer one logical change per commit.
- Do not include issue numbers unless explicitly requested.
- Do not mention implementation details in the title.

## Language

- English is the default language for all generated commit messages.
- Always generate commit messages in English, regardless of the user's language.
- Use clear, concise, and professional English.
- Only generate commit messages in another language if the user explicitly requests it.

## Commit Body

Include a body only when it adds useful context.

Example:

feat(auth): implement password reset

- generate secure reset tokens
- invalidate previous tokens
- add email notification support

## Footer

Include footers only when appropriate.

Examples:

BREAKING CHANGE: Authentication endpoints now require OAuth2.

Co-authored-by: John Doe <john@example.com>

## Multiple Changes

If the staged changes contain unrelated work:

- Do not generate a single commit.
- Explain that multiple commits are recommended.
- Generate one commit message for each logical change.

## Output Requirements

Return only the commit message.

Do not explain your reasoning.

Do not use Markdown.

Do not wrap the output in code blocks.
