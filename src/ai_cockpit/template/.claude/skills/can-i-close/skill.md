# /can-i-close — Pre-Close Contract Check

## When to Use
Before `/land`. Before closing a session. When you want to know if it's safe to stop.

## What It Does
Evaluates three contracts and returns a verdict: CLEAR, WARN, or BLOCK.

## The Three Contracts

### Contract 1: Workspace
*Can the machine be left?*

| # | Check | BLOCK if | WARN if |
|---|-------|----------|---------|
| 1 | Uncommitted changes | Files modified with no commit | Untracked files exist |
| 2 | Unpushed commits | Commits ahead of remote | — |
| 3 | Merge conflicts | Unresolved conflict markers | — |
| 4 | Secrets staged | .env, credentials, tokens in git staging | Sensitive-looking files modified |
| 5 | Running processes | Dev servers, builds, watchers still active | Background jobs from this session |
| 6 | CI status | Pipeline failing on current branch | Pipeline pending |
| 7 | Docker state | Containers started this session still running | — |

### Contract 2: Session
*Can the cockpit be parked?*

| # | Check | BLOCK if | WARN if |
|---|-------|----------|---------|
| 8 | Bookmark exists | — | No bookmark yet (first close attempt) |
| 9 | Devlog updated | — | No devlog entry this session |
| 10 | Tasks resolved | Tasks marked in-progress with no update | Tasks created but not started |
| 11 | WIP state | Half-applied migration, partial refactor | Feature branch with no PR |
| 12 | Blocked items documented | Known blocker with no note | — |

### Contract 3: Conversation
*Can the AI be dismissed without losing value?*

| # | Check | BLOCK if | WARN if |
|---|-------|----------|---------|
| 13 | Promises delivered | Explicitly said "I'll do X" and didn't | Discussed doing X, unclear if done |
| 14 | Decisions recorded | Architecture/strategy decision made in chat but not in ADR, devlog, or commit | — |
| 15 | Knowledge captured | Important pattern/finding discussed but not in memory or docs | — |
| 16 | Workflow complete | Multi-step work partially done (built but didn't test, tested but didn't ship) | — |
| 17 | Research persisted | URLs, findings, analysis done but not saved anywhere | — |
| 18 | User questions answered | User asked something that wasn't resolved | — |
| 19 | Bugs found but not filed | Discovered a bug during work, didn't create a task or issue | — |
| 20 | User preferences saved | User gave feedback/correction not yet in memory | — |

## Execution Steps

### 1. Gather State (parallel)

Run in parallel:
- `git status --porcelain` — dirty files
- `git log @{push}..HEAD --oneline 2>/dev/null` — unpushed commits
- `git diff --name-only --cached` — staged files
- Read `state.json` — session watermarks
- Check for running processes: `ps aux | grep -E 'node|python|ruby|cargo|go run' | grep -v grep` (lightweight, skip if not applicable)

### 2. Evaluate Workspace Contract

Check each item against git state and process list. Score as CLEAR/WARN/BLOCK.

### 3. Evaluate Session Contract

Check bookmark existence (`~/.claude/bookmarks/`), devlog state, task state from backlog or ike if available.

### 4. Evaluate Conversation Contract

Review the conversation history in this session:
- Scan for phrases like "I'll", "let me", "next I'll", "TODO" that indicate promises
- Check if decisions were discussed that aren't in any persistent store
- Look for research/findings that only exist in chat context
- Check if user gave corrections or preferences not saved to memory

### 5. Report

Output format:

```
  ┌─────────────────────────────────────────┐
  │           CAN I CLOSE?                  │
  └─────────────────────────────────────────┘

  WORKSPACE ··········· CLEAR | WARN | BLOCK
  SESSION ············· CLEAR | WARN | BLOCK
  CONVERSATION ········ CLEAR | WARN | BLOCK

  ─── BLOCKS ──────────────────────────────
  (list each BLOCK item with what to do)

  ─── WARNINGS ────────────────────────────
  (list each WARN item — safe to ignore but you'll lose something)

  ─── VERDICT ─────────────────────────────
  CLEAR → "Safe to /land."
  WARN  → "Safe to /land, but you'll leave loose ends."
  BLOCK → "Not safe. Fix these first, or /land --force to override."
```

## Rules

- **Be honest, not paranoid.** A BLOCK means real work loss. A WARN means nice-to-have. Don't BLOCK on trivial items.
- **Conversation contract is the hardest.** You're auditing yourself. Be rigorous — scan the actual conversation, don't just say "looks fine."
- **Fast.** Target 3-4 tool calls. Don't run tests or builds — just check state.
- **Actionable.** Every BLOCK and WARN must say what to do about it.
- **No false CLEARs.** If you're unsure, WARN. Only CLEAR when you've actually checked.
- **This skill does NOT land.** It reports. The pilot decides. Then `/land` executes.
