# 05: Full-season baseline backtest and report skeleton

**What to build:** The tracer bullet extended across a whole held-out season. Every evaluable gameweek is projected, solved and scored for the recent-form baseline arm and the hindsight ceiling, with results recorded per gameweek and aggregated. A report module writes the tables in the repository's existing tabular convention, saves artefacts to disk so a run can be reviewed without repeating it, and prints the known limitations alongside the numbers.

At the end of this ticket there is a complete, trustworthy reference measurement for the incumbent baseline — before any modelling work begins. That ordering is deliberate: it means a later model failure is attributable to the model rather than to the harness.

**Blocked by:** 04: Baseline arm versus hindsight ceiling, one gameweek

**Status:** ready-for-agent

- [ ] All evaluable gameweeks of the held-out season run from a single command
- [ ] Gameweeks that cannot be evaluated are skipped and reported as skipped, so no aggregate is silently diluted
- [ ] Double gameweeks are handled correctly and flagged in the output, since additive points across two fixtures make those weeks structurally higher-scoring
- [ ] Per-gameweek and aggregate results are produced for every arm
- [ ] The per-gameweek table shows coherent variation rather than a flat line, so a non-varying projection is visible by inspection
- [ ] Tables match the repository's existing tabular formatting convention
- [ ] Results and artefacts are written to disk under a single output root
- [ ] The report prints the limitations that constrain interpretation, so numbers cannot be quoted without their caveats
- [ ] A narrowing option allows a partial run over a few gameweeks for smoke testing
- [ ] Aggregate totals are reconciled against the sum of per-gameweek values, so no rows are silently dropped
