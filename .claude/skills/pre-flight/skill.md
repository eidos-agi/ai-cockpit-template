# /pre-flight — Quick Situational Scan

## When to Use
- Called inline by `/takeoff` (not a subagent — runs in main context)
- Can also be called independently mid-session for a status refresh

## What It Does
Builds a fast situational briefing from commit messages, bookmark state, and CLAUDE.md. This is not a deep scan — it's a quick synthesis of signals already available.

## Tone

**Brief. Decisive. No fluff.** Write like a pilot scanning instruments before departure, not a consultant writing a report.

## Execution

### Data Sources (use what's already in context)

1. **Recent commits** — the last 5-10 commit messages on the current branch (from `gitStatus` or fleet sync). These are the single best signal for what happened recently.
2. **Bookmark** — last session summary, next actions, blockers, lifecycle state (from takeoff's Step 1).
3. **CLAUDE.md** — status sections, active workstreams, rules (already loaded in session context).
4. **Fleet data** — if fleet sync ran, notable commits from other repos and fleet health. Skip if no fleet.

**Do NOT make extra tool calls.** If the data isn't already in context, skip that section.

### Output Format

```
RECENT ACTIVITY
  <One-line commit messages, most recent first>
  <If fleet: include notable cross-repo commits>

RESUME
  <Bookmark summary in plain English>
  <Or "First session" / "No bookmark">

NEXT
  1. <Most impactful action — and WHY>
  2. <Second priority>
  3. <Third priority>

BLOCKERS
  <What's stuck and why, or "None">
```

### Quality Check

Before returning:
- [ ] Is every line derived from actual data (commits, bookmark, CLAUDE.md)?
- [ ] Are the NEXT items actionable session goals, not vague backlog items?
- [ ] Did you avoid padding with noise? If a section has nothing meaningful, say "None" or skip it.
- [ ] Zero extra tool calls?

## Rules
- **No scanning.** No file reads. No subagents. Only use data already in context.
- **Commit messages are truth.** They tell you what actually happened, not what was planned.
- Skip sections with no data. Don't show empty sections.
- If called independently (not from takeoff), you may read `state.json` and run `git log --oneline -10` — but only those two calls.
- Prioritize recency. Most recent activity first.
- Flag staleness. Anything from 7+ days ago gets a "(stale)" marker.
