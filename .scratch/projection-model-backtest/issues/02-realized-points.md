# 02: Realized points from actual outcomes

**What to build:** A pure function that converts a solver squad selection plus a loaded actuals frame into the points that squad genuinely would have scored in that gameweek, under real FPL scoring rules. It reads the solver's own selection output — the starting eleven, the ordered bench, the captain and vice-captain flags, position types, and the multiplier the solver already applies — and resolves automatic substitutions and captaincy fallback exactly as the game does. One identical code path is used for every arm of the backtest, so competing projection sources are scored like-for-like.

This is the module every Layer 2 number depends on. Its rules are fiddly and an error here produces no error at all — just quietly wrong totals.

**Blocked by:** 01: Load historical seasons into a tidy frame

**Status:** ready-for-agent

- [ ] A bench player substitutes for a non-playing starter when the resulting formation remains legal, and their points are counted
- [ ] No substitution occurs when the only available substitution would make the formation illegal, and the bench points are not counted
- [ ] The substitute goalkeeper replaces the starting goalkeeper when the starter does not play
- [ ] The captain plays zero minutes, so the vice-captain's points are doubled instead
- [ ] The captain plays, so the vice-captain's points are not doubled
- [ ] When all eleven starters play, no substitution occurs and the bench contributes nothing
- [ ] The solver's own multiplier is respected, so a triple-captain week is scored at triple weight
- [ ] The result reports per-player contributions alongside the total, so any discrepancy can be traced to a player
- [ ] The function is pure: same inputs always produce the same total, with no I/O and no mutation of its inputs
