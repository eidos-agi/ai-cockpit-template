# Learning Browser — Cockpit Plugin

Persistent browser research sessions using [agent-browser](https://github.com/anthropics/agent-browser), customized per cockpit.

## Plugin Pattern

This is a **plugin**, not a built-in. Each cockpit forks this directory and customizes it for their domain:

- A finance cockpit bookmarks bank portals and stores categorization learnings
- A security cockpit maintains authenticated sessions to vulnerability databases
- A planning cockpit researches vendors and stores competitive intel

The plugin provides the skeleton. You fill in the domain knowledge.

## Setup

1. **Install agent-browser** globally (`npm i -g @anthropic/agent-browser`)
2. **Create profile**: `mkdir -p .agent-browser-profile && echo ".agent-browser-profile/" >> .gitignore`
3. **Add to CLAUDE.md** — tell the cockpit how to use it (see below)

## CLAUDE.md Addition

```markdown
## Browser Research
- **ALWAYS use `agent-browser` CLI via Bash** for web research
- Profile: `--profile <cockpit-root>/.agent-browser-profile --headed`
- First call: include `--profile` and `--headed` (daemon remembers until `close`)
- Commands: `open <url>`, `snapshot -i`, `click @ref`, `fill @ref "text"`, `screenshot [path]`, `close`
```

## Persistent Sessions

The agent-browser daemon stays alive until `close` is called. Cookies persist in the profile directory.

- **Don't call `close`** unless you're done with that site
- **Next session**: `agent-browser snapshot -i` to see where you left off
- **After machine restart**: re-open with `--profile` — cookies restore most sessions
- **For 2FA sites**: log in manually once, the profile remembers

## Customization

Fork this directory and add:

```
tools/learning-browser/
├── README.md              ← customize for your domain
├── sites.md               ← bookmarked sites + login notes (gitignored if sensitive)
└── learnings/             ← research findings stored as markdown
    ├── 2026-03-10-banks.md
    └── ...
```

The `learnings/` directory is yours. Store whatever your domain needs — research notes, screenshots, competitive analysis, vendor comparisons.
