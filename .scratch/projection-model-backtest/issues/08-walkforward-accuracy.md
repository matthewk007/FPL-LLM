# 08: Walk-forward predictions and accuracy metrics

**What to build:** A single command that evaluates the model across every evaluable gameweek of the held-out season, retraining as it goes. For each gameweek it trains on all prior seasons plus earlier gameweeks of the test season, predicts that gameweek, and records accuracy — matching how the model would actually be deployed rather than a single frozen fit.

Metrics are reported per gameweek and aggregated: point error across the whole pool and restricted to players who actually scored, per-gameweek rank correlation over both the whole pool and players who played, minutes error, and a reliability table with a Brier score for the appearance probability. The played-only rank correlation is the decision-relevant one, because selection happens among players likely to feature while the whole-pool figure mainly measures inability to rank non-players.

Projections are emitted in a form the next ticket can feed to the solver.

**Blocked by:** 07: Two-stage projection model

**Status:** ready-for-agent

- [ ] Every evaluable gameweek of the held-out season is evaluated, retraining at each one
- [ ] Training for a gameweek uses only prior seasons and earlier gameweeks of the test season
- [ ] Gameweeks with no usable prior history are skipped and reported as skipped
- [ ] Point error is reported for the whole pool and for players who scored
- [ ] Per-gameweek rank correlation is reported for the whole pool and for players who played
- [ ] Minutes error is reported
- [ ] A reliability table and Brier score are reported for the appearance probability, so a stated chance can be checked against how often it happened
- [ ] All metrics are available per gameweek and in aggregate
- [ ] Projections for each gameweek are written in a form the solver can consume directly
- [ ] Point error sits in the low single digits on a smoke run — a wildly different value indicates a join or leakage bug and should be investigated as one before being read as a modelling result
