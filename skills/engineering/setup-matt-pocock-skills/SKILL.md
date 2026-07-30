---
name: setup-matt-pocock-skills
description: Configure the current OpenCode repo for the engineering skills — set up its issue tracker, triage label vocabulary, and domain doc layout. Run once before first use of the other Matt Pocock engineering skills.
metadata:
  author: Matt Pocock (adapted for OpenCode)
  license: MIT
  version: 1.0.0
---

# Setup Matt Pocock's Skills for OpenCode

Scaffold the per-repo configuration that the engineering skills assume:

- **Issue tracker** — where issues live (GitHub by default; local markdown also supported)
- **Triage labels** — the strings used for the five canonical triage roles
- **Domain docs** — where `CONTEXT.md` and ADRs live

This is a prompt-driven skill, not a deterministic script. Explore, present what you found, confirm with the user, then write.

## Process

### 1. Explore

Look at the current repo to understand its starting state:

- `git remote -v` — is this a GitHub repo? Which one?
- `CONTEXT.md` and `docs/adr/` at the repo root
- `.scratch/` — sign that a local-markdown issue tracker convention is already in use
- Is the `triage` skill available? This decides whether triage labels are needed.
- `opencode.json` — check existing config

### 2. Present findings and ask

Summarise what's present and what's missing. Then take the sections in order — one section, one answer, then the next.

**Section A — Issue tracker.**

Default posture: these skills were designed for GitHub. If a `git remote` points at GitHub, propose that. Otherwise offer:

- **GitHub** — issues live in the repo's GitHub Issues (uses the `gh` CLI)
- **GitLab** — issues live in the repo's GitLab Issues (uses the `glab` CLI)
- **Local markdown** — issues live as files under `.scratch/<feature>/` in this repo
- **Other** (Jira, Linear, etc.) — describe the workflow in one paragraph

Record the choice in `docs/agents/issue-tracker.md`.

**Section B — Triage label vocabulary.** Skip if `triage` skill isn't available.

Ask: "Do you want to keep the default triage labels? (recommended: **yes**)"

Defaults: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`.

**Section C — Domain docs.** Default to **single-context** — one `CONTEXT.md` + `docs/adr/` at the repo root.

### 3. Confirm and edit

Show the user a draft of what will be written. Let them edit before writing.

### 4. Write

Write the following files:

**`docs/agents/issue-tracker.md`** — configuration for which issue tracker to use.

**`docs/agents/triage-labels.md`** (only if triage is available) — mapping of canonical role names to label strings.

**`docs/agents/domain.md`** — domain doc consumer rules and layout.

If `CONTEXT.md` doesn't exist, create an empty one with a placeholder.

### 5. Done

Tell the user the setup is complete and which engineering skills will now read from these files. Mention they can edit `docs/agents/*.md` directly later.
