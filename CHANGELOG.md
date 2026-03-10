# Changelog

## v1.3.0 — 2026-03-10

Lean takeoff, fleet sync, phone-home, and learning browser plugin.

### Takeoff — Rewritten for Speed
- **No more subagent.** Pre-flight runs inline from data already in context (commits, bookmark, CLAUDE.md). Max 3 tool calls for entire boot.
- **Commit messages are the briefing.** Recent Activity section shows last 5-10 commits — the actual truth of what happened.
- **HTML is opt-in.** `cockpit.html` only generated when explicitly requested or `custom.html_dashboard` is true.
- **Phone Home (Step 1.5)** — Runs `./bin/update-from-template --check` silently. Surfaces available updates in status bar. Never blocks.
- **Fleet Sync (opt-in Step 0)** — Only runs if `cockpit.org` or `cockpit.repos_dir` is set. Discovers repos via `gh`, fetches/pulls, builds pilot activity maps. Single-repo cockpits skip entirely.

### Pre-flight — Now Lightweight
- Runs inline (no subagent, no Explore agent)
- Sources: commit messages, bookmark, CLAUDE.md, fleet data (if available)
- Zero extra tool calls — synthesizes from session context only
- Format simplified: RECENT ACTIVITY → RESUME → NEXT → BLOCKERS

### Cockpit HTML Template
- Added fleet dashboard CSS (repo tables, pilot activity cards, status colors)
- Added `{{FLEET_SECTION}}` placeholder
- **Now opt-in** — not generated unless requested

### New Skill
- `/clean-sweep` — Workspace-wide commit, push, build, test sweep

### New Plugin: Learning Browser
- `tools/learning-browser/` — plugin pattern for persistent browser research
- Each cockpit forks and customizes for their domain (finance, security, planning, etc.)
- agent-browser profile per cockpit, session persistence across days

### State
- state.json template documents optional fleet fields (`org`, `repos_dir`, `fleet`, `pilots`)
- Added `cockpit.template` and `cockpit.template_version` for phone-home

## v1.2.1 — 2026-02-26

### Tooling
- `bin/update-from-template` — Pull-based skill sync for downstream cockpits. Compares local skills against upstream manifest via SHA256 hashing, won't clobber local customizations.
- Fixed bash arithmetic bug in `update-from-template` (increment helper for `set -e` compatibility)

### Manifest
- `skills_manifest.json` now generated with `bin/generate-manifest` — SHA256 hashes for all template skills

## v1.2.0 — 2026-02-26

### Tooling
- `bin/sync-skills` — Push template skill updates to all downstream cockpits listed in `bin/cockpit-registry.txt`
- `bin/cockpit-registry.txt` — Registry of local cockpit paths for push-based sync
- `bin/generate-manifest` — Generate `skills_manifest.json` with SHA256 hashes

### Documentation
- Added sync-skills instructions to README
- Updated "In the Wild" section with production cockpits

## v1.1.0 — 2026-02-25

Upstream improvements from production use across AIC Director and Greenmark cockpits.

### Takeoff
- Reuse `gitStatus` from session context instead of re-fetching (faster boot)
- ASCII art header for visual identity
- Handle corrupted bookmark files gracefully ("starting fresh" instead of crashing)

### Land
- Three invocation modes: interactive (`/land`), scripted (`/land <debrief>`), silent (`/land clean`)
- ASCII art header for visual identity
- Max 2 tool calls for faster exit

### New Skills
- `/cockpit-repair` — validate state.json, bookmarks, and skills; offer fixes for corruption

### State
- Added `template` field to `cockpit` section in state.json
- Bumped version to 1.1.0

## v1.0.0 — 2026-02-25

Initial release.

### Skills
- `/takeoff` — Boot sequence with bookmark resume and drift detection
- `/land` — Park sequence with outcome capture and bookmark write
- `/cockpit-status` — Instrument panel for active work, blockers, ages

### State Management
- `state.json` with watermarks, counters, and extensible `custom` key
- Bookmark schema v1.1 written to `~/.claude/bookmarks/`

### Documentation
- README with quick start, architecture, design principles
- Customization guide with examples (meeting-driven, email-driven, code-driven cockpits)
- CLAUDE.md template with placeholder sections
