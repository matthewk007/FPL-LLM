# 10: Consolidated report and live-path regression

**What to build:** The final deliverable — a single consolidated result stating the headline comparison plainly enough to act on, with the supporting detail behind it. It brings together prediction accuracy, solver-level realized points for all arms, the regret against the ceiling, and the per-gameweek spread. Every limitation that constrains interpretation is printed with the conclusion, including that the external publisher could not be compared, that the baseline is a reconstruction, that the historical setup is approximate, and that one season carries real variance.

It closes with a regression check that the feature has not disturbed the live weekly workflow: the existing command is re-run and must reproduce the same plan as before this work.

The ticket exists because the point of the whole exercise is a decision — whether to trust the model over the current source — and that decision needs a defensible written conclusion, not a directory of tables.

**Blocked by:** 09: Model arm joins the solver backtest

**Status:** ready-for-agent

- [ ] A consolidated result presents prediction accuracy and solver-level realized points for all arms in one place
- [ ] The headline conclusion is stated in prose, not left for the reader to infer from tables
- [ ] Per-gameweek spread is shown alongside totals, so consistency is visible
- [ ] All interpretation-constraining limitations are printed with the result, including that one season carries real variance and a second season should confirm any positive finding
- [ ] The report states that comparison against the external publisher was not possible and why
- [ ] Baseline quality is reported honestly: the share of player-gameweeks served by archived values versus reconstruction, and the measured difference between the two where they overlap
- [ ] Double gameweeks are called out in the per-gameweek results, so a high-scoring week is not mistaken for model skill
- [ ] The full experiment is reproducible from the documented configuration, with one command per phase
- [ ] The live weekly command is re-run and reproduces the same plan as before this work
- [ ] Linting and formatting pass under the repository's existing configuration
- [ ] The total runtime of a full experiment is documented, so future iterations are planned realistically
