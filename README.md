# Migration World Cup League

An office sweepstake tracker for the 2026 FIFA World Cup. 14 colleagues each own nations drawn across strength tiers; the app scores their results, tracks the knockout bracket, and crowns a leaderboard — all from real tournament results.

## Usage

It's a single self-contained HTML file — no build step, dependencies, or server required.

- **Open directly:** double-click `Migration_World_Cup_League_21.html`, or
- **Serve locally** (recommended, some browsers restrict `file://`):
  ```
  python3 -m http.server 8000
  # then open http://localhost:8000/Migration_World_Cup_League_21.html
  ```

State (scores, stages, knockouts) persists in the browser via `localStorage`.

## Scoring

| Event | Points |
|-------|--------|
| Group-stage win | +3 |
| Group-stage draw | +1 |
| Reach Round of 32 | +5 |
| Reach Round of 16 | +8 (cumulative) |
| Reach Quarter-final | +12 (cumulative) |
| Reach Semi-final | +18 (cumulative) |
| Reach Final | +25 (cumulative) |
| Win the World Cup | +40 (cumulative) |
| Giant-Killer (beat a higher-tier nation) | +5 |

Group results through the end of the group stage are baked in and reflect the real 2026 World Cup. Log knockout results as they happen in the **Rosters & Scoring** tab.

See [CLAUDE.md](CLAUDE.md) for the architecture and data model.
