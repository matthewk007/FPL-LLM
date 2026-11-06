# 06: Feature construction with the leakage guard

**What to build:** The model input matrix, built strictly from information available *before* the gameweek being predicted. Lags are computed within each season: rolling recent points across several windows, rolling minutes and a start rate, price level and price momentum, and rolling team strength for a player's own team and their opponent, derived from goals scored and conceded. Home/away is explicit. Optional underlying statistics carry an accompanying presence indicator, and missing values are imputed by position median rather than dropped, so the feature vector keeps a fixed width across seasons.

The centrepiece is a single reusable assertion: features for gameweek N can be rebuilt with all rows at or after N removed, and the gameweek-N rows must come out identical. A feature that accidentally incorporates future information still produces a well-formed frame and a plausible result, so nothing downstream can detect it — this is the check that catches the most dangerous class of bug in the project.

Cross-season player identity is deliberately not attempted here; each season is treated as an independent series.

**Blocked by:** 01: Load historical seasons into a tidy frame

**Status:** ready-for-agent

- [ ] Features exist for rolling points over multiple windows, rolling minutes, and start rate
- [ ] Price level and price momentum are available as features
- [ ] Home/away is an explicit feature
- [ ] Lagged team strength is available for both the player's own team and the opponent
- [ ] Optional underlying statistics are used when present, with a presence indicator and position-median imputation when absent
- [ ] The feature vector has identical width and column order across all seasons, regardless of which optional statistics exist
- [ ] The leakage guard is exposed as a single assertion that can be run for any gameweek
- [ ] Degraded features in the first gameweeks of a season are produced without error, as the real cold-start condition
- [ ] No feature for a gameweek is derived from any row of that gameweek or later
