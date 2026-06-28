# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single, self-contained HTML file (`Migration_World_Cup_League_21.html`) implementing an office sweepstake tracker for the 2026 FIFA World Cup. 14 colleagues ("contestants") each own nations drawn across strength tiers; the app scores their results, tracks the knockout bracket, and crowns a leaderboard. There is **no build system, no dependencies, no tests, and no package manager** — everything (HTML, CSS, JS, an embedded base64 trophy image) lives in the one file.

## Running / previewing

Open the HTML file directly in a browser, or serve the directory and load it (some browsers restrict features on `file://`):

```
python3 -m http.server 8000   # then open http://localhost:8000/Migration_World_Cup_League_21.html
```

To verify changes, load it in a browser and exercise the tabs; check the devtools console for errors. There is no automated test suite.

## Working with the file

- **Lines are extremely long.** The JS is written one-statement-per-line with minified-style density, and there is a ~1MB base64 image near the top. Reading the whole file will blow the token limit. Use ranged reads or `awk 'NR>=A && NR<=B {print NR": "substr($0,1,N)}'`, and locate code by **symbol name** (`grep -n 'function foo'`) rather than by line number, since edits shift line numbers.
- For multi-line edits inside template literals (lots of backticks and `${}`), prefer a small Python find-and-replace script over hand-matching whitespace.

## Architecture

All app logic is in one `<script>` block starting at `const KEY="mwc_league_v2"`. (There are separate small inline scripts for export/offline-recap iframes — distinct from the main app.)

### Data lives in top-of-script constants
`PLAYERS`, `TIERS` (A/B/C tiers of 14 + D = 6-team "Cinderella" pot), `LOCKED_ASSIGNMENTS` (who owns which nation), `COUNTRY_DATA`, `FLAGS`, `PLAYER_BIOS`, `MATCH_PREVIEWS` (all 72 group fixtures), `SEEDED_RESULTS` (baked-in scores), `STAGES`/`STAGE_CUM` (knockout stage bonuses).

### The draw is permanently locked
`enforceLockedDraw()` / `lockedStateFrom()` rebuild state from `LOCKED_ASSIGNMENTS` on every load, keyed by `LOCKED_DRAW_VERSION`. The live-draw ceremony code (`preDrawPanel`/`drawingPanel`) still exists but is bypassed. **To change ownership, edit `LOCKED_ASSIGNMENTS` and bump `LOCKED_DRAW_VERSION`.**

### Two parallel scoring systems (important)
1. **`state.scores[team]`** = `{w, d, stage, gk, out}` — drives sweepstake points via `teamPts()` (`w*3 + d*1 + STAGE_CUM[stage] + gk*5`) and the **leaderboard** (`leagueStandings`, `playerTotal`).
2. **`groupStandings()`** — recomputed independently from `state.matchStates` (per-match scores), drives the **group tables** and **bracket qualification**.

These are kept in sync by the seeding/sync functions below. When adding results, both systems must end up consistent.

### Results seeding
`SEEDED_RESULTS` (array of `{key, aScore, bScore, winner|draw, note}`, where `key` = `matchKey()` = `date|group|a|b`) is applied by `applyResultSeeds()`, gated on `RESULT_SEED_VERSION`. It only writes matches that are still "untouched" and increments `state.scores` w/d at the same time. **To push new/changed results to already-saved states, bump `RESULT_SEED_VERSION`** — otherwise returning users keep their old data.

### Knockout bracket & qualification
- `R32_TIES` describes the 16 Round-of-32 ties using slot codes: `1X`/`2X` = winner/runner-up of Group X; `3:GROUPS` = a best-third slot whose candidate groups are listed.
- `slotTeam()`/`slotInfo()` resolve a slot to a concrete team **only once that group is complete** (`groupComplete()`); otherwise they render a "Winner/Runner-up/Best 3rd" placeholder.
- `THIRD_PLACE_QUALIFIERS` maps each third-place slot string to the specific group whose 3rd-placed team fills it (the app does not compute FIFA's best-third allocation table at runtime — it's a fixed, pre-resolved mapping).
- `confirmedQualifiers()` is the source of truth for "who's through". On load, `init()` runs `syncQualificationStages()` (awards the +5 R32 stage bonus to qualifiers) and `syncGroupEliminations()` (marks every non-qualifier in a completed group `out`). `eliminatedTeams()` derives the landing-page "graveyard" from the same logic.

### Rendering & state
- `currentTab` + `render()` route to panel builders (`doneDrawPanel`, `awardsPanel`, `rostersPanel`, `boardPanel`, `matchesPanel`, `groupsPanel`, `rulesPanel`). Each panel returns an HTML **string** built from template literals; `render()` injects it and then calls `wire()` to (re)bind all event handlers. There is no framework — after any state change you call `save(); render();`.
- The **Home/landing tab** (`data-tab="draw"`) is `doneDrawPanel`: it shows the full knockout bracket plus the eliminated-teams graveyard. The sticky R32 strip (`knockoutStripLanding` in `#koBandHost`) is shown on every tab *except* Home (where the full bracket already appears).
- **Persistence** (`load`/`save`, `KEY="mwc_league_v2"`): tries shared `window.storage` first, then `localStorage`, then in-memory. All user edits (scores, stages, out-flags via the Matches and Rosters tabs) persist here and survive reloads.

### Scoring rules (encoded, not just documented)
Group win +3, draw +1; cumulative knockout bonuses in `STAGE_CUM` (R32 +5 → Champion); Giant-Killer/upset = `gk` count × 5. Editing the rules text in `rulesPanel` does **not** change scoring — change `teamPts()` / `STAGE_CUM` for that.
