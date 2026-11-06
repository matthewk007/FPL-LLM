# 04: Baseline arm versus hindsight ceiling, one gameweek

**What to build:** The first complete path through every layer, for a single gameweek. A projection is emitted for the whole player pool using the baseline, solved into a squad, and scored against actual outcomes. A second arm is built by projecting *actual* outcomes, giving the best squad hindsight could have assembled that week — the ceiling. Both are printed side by side with the gap between them.

This is the tracer bullet: it deliberately needs no trained model, and it proves data loading, projection emission, historical solving, realized scoring, and reporting all connect. The number it prints is the reference every later result is measured against.

The baseline has two sources, in priority order, because the dataset ships **real archived expected points** for a subset of gameweeks: use the archived value where it exists for that gameweek and player, and reconstruct from recent scoring only where it does not. Archived values were measured as clearly more accurate than the reconstruction on the overlap — 1.498 versus 1.954 mean absolute error at GW9 of the test season — so preferring them is a measured decision, not a convenience.

**Blocked by:** 02: Realized points from actual outcomes; 03: Single-gameweek historical solve

**Status:** ready-for-agent

- [ ] A single gameweek is projected, solved, and scored end to end with one command
- [ ] The baseline prefers archived expected points for that gameweek and player, and falls back to reconstruction only where no archived value exists
- [ ] The output records, per player, which baseline source was used, so the mix is auditable
- [ ] A hindsight-optimal arm is produced by projecting actual outcomes through the identical pipeline
- [ ] Both arms are scored by the same realized-points path, so the comparison is like-for-like
- [ ] Output shows each arm's total and the regret between them, plus the selected squads
- [ ] The projection file emitted uses the schema the solver already consumes, and its name cannot be mistaken for the external publisher's files by the live loader's filename pattern
- [ ] Exactly one gameweek column is emitted, since the solver aggregates expectation across every projection column present
- [ ] Where an archived value and a reconstruction both exist, their difference is recorded so reconstruction error is measured on real data rather than assumed
- [ ] The chosen baseline gameweek is one where archived values exist, so the first end-to-end run is exercised against the most trustworthy reference
