---
name: mcp-setup
description: Configure MCP servers (Context7, GitHub, Playwright, Figma, and more) for Claude Code, Cursor, Windsurf, or OpenCode. Provides exact commands, config snippets, and setup validation steps.
metadata:
  origin: custom
  version: 1.0.0
---

# MCP Setup

Configure Model Context Protocol servers to give AI agents live access to documentation, repositories, browsers, design files, and more.

## When to Activate

- Setting up a new development environment
- Adding MCP servers to an existing config
- User asks about MCP configuration
- Debugging MCP connection issues
- User wants Context7, GitHub, Playwright, or Figma integration

## Top 4 MCP Servers

### 1. Context7 — Version-Accurate Documentation

Stops hallucinated APIs by fetching real, version-pinned docs.

**Install (all clients):**
```bash
npx -y @upstash/context7-mcp
```

**Add to OpenCode/Claude Code (.mcp.json):**
```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    }
  }
}
```

**Usage:** Append `use context7` to any prompt:
> "Write a React 19 server component with Server Actions, use context7"

**Tool count:** 2 (compact by design)

---

### 2. GitHub MCP (Official) — Repository, PR, and Issue Access

Full GitHub API access: read repos, open PRs, manage issues, trigger Actions.

**Install:**
Requires a GitHub Personal Access Token with `repo` and `workflow` scopes.

**Add to config:**
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_..."
      }
    }
  }
}
```

**Tool count:** ~20 (remote hosted), ~46 (Docker self-hosted)

**Best for:** Any team whose work touches GitHub.

---

### 3. Playwright MCP (Microsoft Official) — Browser Automation

Navigate pages, click, fill forms, take screenshots, run JavaScript in a real browser.

**Install:**
```bash
npx -y @playwright/mcp
```

**Add to config:**
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp"]
    }
  }
}
```

**Tool count:** ~20 (navigation, click, input, screenshot)

**Best for:** Visual testing, UI debugging, scraping JS-heavy pages, agentic web flows.

---

### 4. Figma MCP (Official) — Design-to-Code Bridge

Pull design context, variables, components, and layout from Figma frames.

**Requirements:**
- Figma Desktop app open in Dev Mode
- Frame selected in the editor
- Figma Personal Access Token

**Add to config:**
```json
{
  "mcpServers": {
    "figma": {
      "command": "npx",
      "args": ["-y", "@anthropic/figma-mcp-server"],
      "env": {
        "FIGMA_ACCESS_TOKEN": "figd_..."
      }
    }
  }
}
```

**Best for:** Converting designs to code, matching UI specs precisely.

---

## Quick-Start: 3-Server Setup (Highest Impact)

For most developers, these three cover 80% of workflows:

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp"]
    }
  }
}
```

---

## Verification Steps

After adding a server:

1. **Restart** the AI client (Claude Code, Cursor, OpenCode)
2. **Test the connection:**
   - Context7: ask "what version of React is current, use context7"
   - GitHub MCP: ask "list my recent PRs"
   - Playwright: ask "take a screenshot of https://example.com"
   - Figma: ensure Figma Desktop is open with a frame selected
3. **Check logs** if it fails:
   - Claude Code: `~/.claude/logs/`
   - Cursor: `~/.cursor/logs/`

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| Server not found | Missing `npx` or Node.js | `node --version` — need v18+ |
| Auth error | Token expired or wrong scopes | Regenerate token with correct scopes |
| Tool not appearing | Config file in wrong location | Check `.mcp.json` is in project root or `~/.claude/` |
| Context7 returns old data | Cache stale | Add `?refresh=true` or wait 5 min |
| Playwright fails on headless | Missing browser binaries | `npx playwright install chromium` |

## Tool Ceiling Warning

- **Cursor** hard limit: 40 tools total across all MCP servers
- **Claude Code** practical limit: ~50 tools before accuracy drops
- **Windsurf** limit: 100 tools

Prefer compact servers (Context7 = 2 tools) over full-surface ones (GitHub Docker = 46 tools) on Cursor.
