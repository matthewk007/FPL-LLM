# 09: Model arm joins the solver backtest

**What to build:** The trained model's projections run through the identical backtest path as the baseline, so all three arms — model, recent-form baseline, and hindsight ceiling — are scored by the same realized-points code over the same gameweeks. Results report each arm's per-gameweek and total realized points plus the regret between the model and the ceiling.

The regret figure is what makes the result interpretable: it defines "good" relative to what was actually achievable in each gameweek rather than in the abstract. Reporting per-gameweek alongside totals matters because a model that wins on average but loses badly in specific weeks is a different proposition from one that is consistently close.

**Blocked by:** 05: Full-season baseline backtest and report skeleton; 08: Walk-forward predictions and accuracy metrics

**Status:** ready-for-agent

- [ ] The model arm is solved and scored through the same realized-points path as the baseline, with no arm-specific scoring logic
- [ ] Model, baseline, and hindsight arms are all present in one comparison
- [ ] Regret is reported against the hindsight ceiling, per gameweek and in total
- [ ] Per-gameweek results and aggregate totals are both reported, so week-to-week variance is visible
- [ ] Every arm covers exactly the same set of gameweeks, so the comparison is not distorted by skipped weeks
- [ ] The model is evaluated on a season it never trained on, and this is stated in the output
- [ ] The report makes clear the external publisher is not among the compared arms, and why
