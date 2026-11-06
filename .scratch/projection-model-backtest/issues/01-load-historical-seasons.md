# 01: Load historical seasons into a tidy frame

**What to build:** From the community historical dataset (one directory per season, each holding a per-gameweek frame and a season-level player-attributes file), produce a single tidy frame with one row per player per gameweek. Each row carries the actual outcome — points, minutes — alongside the attributes needed as model inputs and solver constraints: price, position, team, opponent, home/away. Optional underlying statistics (expected goals and assists, bonus, ownership) are used when present and tolerated when absent, so older seasons remain usable for training. Required columns are validated up front and their absence reported in full. Position and team naming are normalised into the solver's vocabulary at this boundary so nothing downstream branches on source-specific spelling.

Note the dataset's real shape, discovered on inspection: the per-gameweek frame lives under a `gws/` subdirectory; `position` and `team` are absent for the two oldest seasons and must be drawn from that season's player-attributes file; and one season (2018-19) is latin-1 encoded while the rest are UTF-8, and has no team-attributes file.

**Blocked by:** None (can start immediately)

**Status:** ready-for-agent

- [ ] Loading a season yields one row per (season, gameweek, player) with no duplicate keys
- [ ] **Double gameweeks are aggregated**: where a player has multiple fixtures in one gameweek, additive statistics (points, minutes, goals, assists, bonus, and similar) are summed and non-additive attributes take a single representative value, so no rows are dropped or duplicated. The test season has double gameweeks in GW26, GW33 and GW36 with 419 affected player-gameweek keys.
- [ ] A missing required column raises an error naming **every** missing column, rather than loading a partial frame
- [ ] Optional underlying-statistic columns are used when present and do not error when absent
- [ ] Position and team values are normalised into the solver's vocabulary on load
- [ ] `position` and `team` are recovered from the season's player-attributes file for seasons whose per-gameweek frame lacks them, and this fallback is documented rather than silent
- [ ] The loader handles the one latin-1 season via an encoding fallback, and does not treat that as a schema failure
- [ ] The frame exposes a stable column set regardless of which optional columns exist
- [ ] Team, opponent, and home/away are available per row
- [ ] A per-season sanity summary is available: player count, price range, and identifier set, suitable for comparison against that season's player-attributes file
- [ ] Types are coerced explicitly on load, so downstream code sees one consistent representation
- [ ] Loading every season in the dataset succeeds, including the partial current one
