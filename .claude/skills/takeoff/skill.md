# /takeoff — Cockpit Boot Sequence

## When to Use
Start of every session. This is the first thing you run.

## What It Does
Reads state + bookmark + recent commits. Shows the header, status bar, and a fast briefing built from commit messages. Writes `takeoff.md`. Waits for orders.

## Execution Steps

### 0. Fleet Sync (opt-in — runs before everything else)

**Skip this step entirely** if `state.json` has no `cockpit.org` and no `cockpit.repos_dir`. Fleet sync is for multi-repo organizations. Single-repo cockpits skip straight to Step 1.

Sync the entire org fleet so takeoff has full situational awareness across all repos.

**A. Discover repos** — run `gh repo list <org> --json name --limit 50` (where `<org>` comes from `state.json → cockpit.org`)
- **Fallback** (if `gh` is unavailable or offline): use the `fleet` map from `state.json` keys + scan `repos_dir` for local directories

**B. For each locally-cloned repo** (path derived from `repos_dir + "/" + repo_name`):
1. `git fetch --all --quiet`
2. `git pull --ff-only --quiet`
3. Capture: current branch, dirty file count, ahead/behind, new-from-remote commit count
4. If pull fails: **warn and continue** — never abort takeoff

**C. Output fleet summary:**
```
  FLEET     <N> repos synced | <N> new commits pulled | <N> unpushed
```

Compact repo table + pilot activity (if `state.json → pilots` exists).

**D. Fallback rules**: `gh` unavailable → scan local dirs. Fetch/pull fails → warn, skip, continue. Network offline → use cached state. **Never abort takeoff for fleet issues.**

### 1. Gather Core State (cheap — 2 reads max)

**A. state.json** — from the cockpit root
- `cockpit.name`, `watermarks.last_land`, `counters.sessions`
- If `last_land` is null, this is the first session ever

**B. Latest bookmark** — one shell call: `ls -t ~/.claude/bookmarks/*-bookmark.json | head -20 | xargs grep -l "<cwd>" | head -1 | xargs cat`
- If found: `lifecycle_state`, `context.summary`, `next_actions`, `blockers`
- If not found: "No previous bookmark"

**C. Git state** — use the `gitStatus` block already injected at session start (or fleet sync data if Step 0 ran). **Do NOT re-run git commands.**

### 1.5. Phone Home (template update check)

**If `bin/update-from-template` exists**, run `./bin/update-from-template --check 2>/dev/null`.

- Update available → show in status bar: `TEMPLATE  update available: v1.3.0 → v1.4.0`
- Up to date or script missing → skip silently. Never block takeoff.

### 2. Output Header + Status Bar

**Cockpit name** as bold Unicode block letters, then:

```
  ████████╗ █████╗ ██╗  ██╗███████╗ ██████╗ ███████╗███████╗
  ╚══██╔══╝██╔══██╗██║ ██╔╝██╔════╝██╔═══██╗██╔════╝██╔════╝
     ██║   ███████║█████╔╝ █████╗  ██║   ██║█████╗  █████╗
     ██║   ██╔══██║██╔═██╗ ██╔══╝  ██║   ██║██╔══╝  ██╔══╝
     ██║   ██║  ██║██║  ██╗███████╗╚██████╔╝██║     ██║
     ╚═╝   ╚═╝  ╚═╝╚═╝  ╚═╝╚══════╝ ╚═════╝ ╚═╝     ╚═╝
```

**Block letter reference** — use this character set for the cockpit name:
```
█ ═ ╗ ╔ ╚ ╝ ║ ╠ ╣ ╦ ╩ ╬ ▀ ▄
```

**Status bar:**
```
  BRANCH    <branch>          DIRTY  <yes (N) | clean>
  SESSION   #<N+1>            LAST   <time since last land>
```

If template update detected: `TEMPLATE  update available: <old> → <new>`
If drift detected: `DRIFT     <what changed since bookmark>`

### 3. Pre-flight Briefing (inline — no subagent)

Build the briefing **directly from what you already have**. No subagent, no extra tool calls.

**Sources** (already gathered):
- Bookmark summary + next_actions + blockers
- Recent commit messages from `gitStatus` (or fleet sync)
- `CLAUDE.md` status sections (already in context)
- Fleet data (if Step 0 ran)

**Compose exactly this:**

```
RECENT ACTIVITY
  <Last 5-10 commit messages from this repo, one line each, most recent first>
  <If fleet sync ran: include notable commits from other repos too>

RESUME
  <Bookmark summary — what happened last session, in plain English>
  <Or "First session" / "No bookmark found">

NEXT
  1. <From bookmark next_actions, or inferred from commits + CLAUDE.md>
  2. <second priority>
  3. <third priority>

BLOCKERS
  <From bookmark blockers, or "None">
```

**This is fast.** No scanning, no subagent, no file reads beyond what's already in context. Just synthesize what you have.

### 4. Generate takeoff.md

Write `takeoff.md` at the cockpit root — the persistent version of the briefing:

```markdown
# <Cockpit Name> — Takeoff #N

**Pilot** <name> | **Date** Mar 10, 2026 | **Branch** `main` | **Dirty** yes/no

> **Resume:** <bookmark summary>

---

## Recent Activity

- `<sha>` <commit message>
- `<sha>` <commit message>
- ...

## Next

1. <priority>
2. <priority>
3. <priority>

## Blockers

<blockers or "None">

---

*Generated <ISO timestamp> by /takeoff*
```

If fleet sync ran, add Fleet Status and Pilot Activity sections between Resume and Recent Activity.

### 5. Generate cockpit.html (only if asked)

**Skip by default.** Only generate `cockpit.html` if the user explicitly requests it or if `state.json` has `"html_dashboard": true` in the `custom` key.

If generating: read `.claude/skills/takeoff/cockpit-template.html`, replace `{{PLACEHOLDER}}` tokens, write to `cockpit.html`.

### 6. Update State

Write to `state.json`:
- Set `watermarks.last_takeoff` to current ISO timestamp
- Increment `counters.sessions` by 1
- Increment `counters.takeoffs` by 1

### 7. STOP

```
Ready for orders.
```

Wait for the user.

## Rules
- **No subagents.** Pre-flight runs inline from data already in context.
- **Max 3 tool calls** for the entire takeoff: read state.json, read bookmark, update state.json. Everything else comes from session context.
- Reuse gitStatus. Never re-fetch what's already there.
- Never ask questions during takeoff. Just show state.
- If no state.json exists, create it and say "First flight. Cockpit initialized."
- If bookmark is corrupted: "Bookmark corrupted — starting fresh" and continue.
- `takeoff.md` is overwritten every takeoff. `cockpit.html` only on request.
- **Fleet sync is opt-in**: only if `cockpit.org` or `cockpit.repos_dir` is set.
- **Phone home is silent**: missing script or failure → skip, don't mention it.
