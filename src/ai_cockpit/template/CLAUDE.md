# CLAUDE.md — AI Cockpit Template

> **Fork this file.** Replace the placeholder sections with your role context, domain rules, and project structure.

## Role Context

<!-- WHO: Replace with your role, org, and mission -->
This is a Claude Code cockpit for **[YOUR ROLE]** at **[YOUR ORG]**.

## Cockpit Primitives

This workspace uses the AI Cockpit Template. Three skills are always available:

| Skill | When | What |
|-------|------|------|
| `/takeoff` | Start of session | Boot sequence: bookmark → drift check → priorities → wait |
| `/land` | End of session | Capture outcomes → write bookmark → clean exit |
| `/cockpit-status` | Anytime | Active workstreams, blockers, ages, ownership |

## Session Protocol

1. Every session starts with `/takeoff`
2. Work using domain-specific skills and tools
3. Every session ends with `/land`
4. If a session crashes without `/land`, the next `/takeoff` will detect drift and flag it

## State Files

- **`state.json`** — Watermarks and counters. Skills read/write this. Committed to git.
- **Bookmarks** — Written to `~/.claude/bookmarks/<session-id>-bookmark.json` by `/land`. Not in git.

## Domain Skills

<!-- LIST: Add your domain-specific skills here -->
| Skill | What It Does |
|-------|-------------|
| `/example-skill` | Placeholder — replace with your skills |

## Project Structure

<!-- STRUCTURE: Replace with your actual directory layout -->
```
your-cockpit/
├── CLAUDE.md          ← you are here
├── state.json         ← session state & watermarks
└── ...                ← your domain structure
```

## Rules

<!-- RULES: Add your domain-specific rules -->
- Soft deletes only — never hard-delete records
- No secrets in repos — use environment variables or a secrets manager
- Commit messages explain why, not just what

<!-- BACKLOG.MD MCP GUIDELINES START -->
<!-- If you use Backlog.md MCP, paste the guidelines block here -->
<!-- BACKLOG.MD MCP GUIDELINES END -->
