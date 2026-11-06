# 03: Single-gameweek historical solve

**What to build:** The ability to run the **real** solver against a past gameweek, by serving it the historical setup it expects instead of current-season data. A historical gameweek's player set, prices, positions and teams are reconstructed from the loaded history, and the fixture list for that gameweek is derived from the same source. Those lookups are substituted inside the backtest process, so the live solver and its run entry point are not touched.

Two traps make this the riskiest integration in the project, and both are handled here. The solver merges projection data against current-season player data, so unreconstructed history silently drops every historical player. And its default expected-minutes threshold is calibrated for multi-gameweek horizons, so at a single-gameweek horizon it would filter the player pool to empty.

**Blocked by:** 01: Load historical seasons into a tidy frame

**Status:** ready-for-agent

- [ ] Given a historical gameweek and a projection file for it, the real solver produces a legal squad: fifteen players within budget and positional limits, a starting eleven, an ordered bench, a captain, and a vice-captain
- [ ] Historical player data and fixtures are served in-process, and no live network call is made for them
- [ ] The resulting player pool is non-empty and contains every player for whom a projection was supplied
- [ ] A pool emptied by filtering is distinguishable from one emptied by a failed join, and reports which of the two occurred
- [ ] Solving a historical gameweek produces a result whose shape matches a live solve, so existing consumers work unchanged
- [ ] No production module is modified — the live weekly run path remains byte-identical apart from the addition of the backtest package
