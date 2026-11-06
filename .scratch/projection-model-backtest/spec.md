# Spec: Projection model + honesty backtest

Status: ready-for-agent

## Problem Statement

The solver's squad quality is bounded entirely by the projection data fed into it. Today the only sources available are a hand-produced spreadsheet from one external publisher, and the naive fallback of recent form. Neither is a model anyone here owns or can explain, and neither can be improved when it proves wrong.

Worse, we currently have no way to know whether a projection source is any good. There is no ground truth anywhere in the repo — no realized points, no accuracy measurement, no scoring rules. Every judgement about whether the solver is doing well rests on an argument rather than a measurement.

The practical consequence: when choosing transfers, we cannot answer "is this projection source better or worse than that one?" with evidence, so decisions get made on intuition, and there is no way to tell a genuine improvement from noise.

## Solution

An offline harness that trains a projection model on historical FPL seasons and measures it honestly against real outcomes, at two layers:

1. **Prediction accuracy** — how close the model's per-player projections are to what players actually scored, including whether it can tell who will be on the pitch at all.
2. **Solver-level outcome** — what a squad built from these projections actually scored, compared against the `ep_next` baseline and against the best squad hindsight would have picked.

Both layers are reported for a held-out season the model never trained on. Nothing in the live solver path changes, and no existing behaviour is altered — this is measurement infrastructure plus one new projection source that can be switched on only if it proves itself.

The user outcome is a defensible answer to "should we trust this model over the current source?", with the numbers and their limitations written down, and the ability to re-run the same measurement whenever either changes.

## User Stories

### Data preparation

1. As a modeller, I want to load historical seasons from a well-known community CSV layout, so that I do not have to hand-build a data pipeline before I can test anything.
2. As a modeller, I want the loader to fail loudly and specifically when a required column is missing, so that a schema mismatch is caught at load time rather than producing silently wrong features twenty steps later.
3. As a modeller, I want the loader to name every column it expected but could not find, so that I can fix the input rather than guess.
4. As a modeller, I want optional columns (underlying statistics such as expected goals and assists) to be used when present and gracefully absent when not, so that older seasons remain usable as training data.
5. As a modeller, I want a stable feature-vector shape regardless of which optional columns exist, so that a model trained on newer seasons can still be applied to older ones.
6. As a modeller, I want the loader to normalise team and position naming into the solver's own vocabulary, so that downstream code never branches on source-specific spelling.
7. As a modeller, I want the tidy frame to carry one row per player per gameweek, so that lagged features are a simple grouped shift rather than a bespoke join.

### Features

8. As a modeller, I want lagged rolling summaries of a player's recent points, so that recent form is available as a feature.
9. As a modeller, I want lagged rolling summaries of recent minutes and a start rate, so that the model can learn availability separately from performance.
10. As a modeller, I want price level and price momentum, so that the market's view of a player is available as a feature.
11. As a modeller, I want rolling team strength for a player's own team and for the opponent, so that fixture difficulty is represented by measured performance rather than a subjective rating.
12. As a modeller, I want home/away as an explicit feature, so that the well-known home advantage is learnable rather than assumed.
13. As a modeller, I want an explicit indicator for whether underlying statistics exist for a row, so that the model can learn to distrust that portion of the feature space on older seasons.
14. As a modeller, I want missing optional values imputed by position median rather than dropped, so that early-season and older-season rows are not silently discarded.
15. As a modeller, I want degraded features in early gameweeks to be a treated condition rather than an error, so that the cold-start problem is measured instead of hidden.
16. As a modeller, I want a single assertion that can prove no feature for gameweek N was derived from any row at or after N, so that the most dangerous class of bug is testable directly.

### Model

17. As a modeller, I want to predict the probability a player appears at all, so that availability is modelled explicitly rather than folded into a single opaque number.
18. As a modeller, I want to predict minutes conditional on appearing, so that the solver receives a meaningful expected-minutes value.
19. As a modeller, I want to predict points conditional on appearing, so that the zero-inflated mass of non-appearances does not dominate the fit.
20. As a modeller, I want expected points to combine appearance probability with conditional points, so that a player who is unlikely to play is projected low even if their per-appearance rate is high.
21. As a modeller, I want expected minutes to combine appearance probability with conditional minutes, so that expected minutes are consistent with expected points.
22. As a modeller, I want the model to be implemented in the libraries already present, so that no new dependency is introduced and the environment stays reproducible.
23. As a modeller, I want each of the three fitted stages to be separately inspectable, so that a bad projection can be traced to availability, minutes, or performance rather than being a single unexplained number.
24. As a modeller, I want the regularisation strength chosen from data, so that it is not a magic number.
25. As a modeller, I want that choice made only within the training data, so that the held-out season stays genuinely held out.
26. As a modeller, I want the fitted model to be serialisable, so that a trained model can be reused without refitting and a run can be reproduced later.

### Walk-forward evaluation

27. As a modeller, I want to train on all prior seasons plus earlier gameweeks of the test season, so that the evaluation matches how the model would actually be deployed.
28. As a modeller, I want to re-train at every evaluation gameweek, so that the measurement reflects expanding information rather than a single frozen fit.
29. As a modeller, I want the test season to be the most recent complete one, so that I get a full season of evaluation points rather than a handful.
30. As a modeller, I want gameweeks that cannot be evaluated (no prior history at all) to be skipped explicitly, so that the denominator of every aggregate is clear.

### Layer 1: prediction accuracy

31. As an evaluator, I want mean absolute error on projected points across the whole player pool, so that overall accuracy has one headline number.
32. As an evaluator, I want mean absolute error restricted to players who actually scored, so that accuracy on the players who matter is not diluted by the large non-playing majority.
33. As an evaluator, I want per-gameweek rank correlation between projected and actual points, so that I know whether the model orders players correctly — which is what selection depends on — rather than only whether its magnitudes are right.
34. As an evaluator, I want that rank correlation reported for the whole pool and for players who played, so that I can distinguish "cannot rank non-players" from "cannot rank footballers".
35. As an evaluator, I want mean absolute error on projected minutes, so that the availability stage is judged on its own terms.
36. As an evaluator, I want a reliability table and a Brier score for the appearance probability, so that I know whether a stated 70% chance actually happens about 70% of the time.
37. As an evaluator, I want metrics reported per gameweek and aggregated, so that I can see whether performance drifts across a season.
38. As an evaluator, I want a flat-lined metric across gameweeks to be visually obvious, so that a projection that is not varying (a common pipeline bug) is caught by inspection.

### Layer 2: solver-level backtest

39. As an evaluator, I want each evaluated gameweek solved from scratch as a legal squad, so that projection quality is isolated from transfer planning.
40. As an evaluator, I want the backtest to use the real solver rather than a re-implementation, so that the measurement reflects the thing actually being shipped.
41. As an evaluator, I want the solver configured for a single gameweek, so that the projection under test is the only thing driving selection.
42. As an evaluator, I want player-pool filtering disabled during the backtest, so that the model is judged on the whole pool rather than a pre-filtered subset.
43. As an evaluator, I want realized points computed by applying real FPL scoring rules to actual outcomes, so that the score is what the squad would genuinely have returned.
44. As an evaluator, I want automatic substitutions applied correctly including the formation constraint, so that a bench player only replaces a starter when the resulting formation remains legal.
45. As an evaluator, I want captaincy to fall back to the vice-captain when the captain did not play, so that the armband is handled as FPL actually handles it.
46. As an evaluator, I want the same realized-points computation applied to every competing projection source, so that the comparison is like-for-like.
47. As an evaluator, I want an `ep_next` baseline arm run through the identical pipeline, so that the model is compared against the incumbent free number rather than against nothing.
48. As an evaluator, I want a hindsight-optimal arm built by projecting actual outcomes, so that there is a ceiling showing how much better any projection could have done.
49. As an evaluator, I want regret expressed as the gap between the model and that ceiling, so that "good" is defined relative to what was achievable rather than in the abstract.
50. As an evaluator, I want totals and per-gameweek breakdowns for all arms, so that a model which wins on average but loses badly in some weeks is visible.

### Reporting

51. As a reader, I want all results in tables matching the repo's existing tabular conventions, so that the output is familiar and skimmable.
52. As a reader, I want results also written to artefacts on disk, so that a run can be reviewed after the fact without re-running it.
53. As a reader, I want a readable summary alongside the tables, so that the conclusion is stated rather than left to the reader to derive.
54. As a reader, I want the known limitations printed with the results, so that the numbers cannot be quoted without their caveats.
55. As a reader, I want a clear statement that the comparison against the external spreadsheet publisher is not part of this backtest, so that nobody mistakes the two-way comparison for a three-way one.

### Operational

56. As a maintainer, I want no changes to the solver or the live run path, so that upstream updates remain easy to pull and nothing ships until the evidence justifies it.
57. As a maintainer, I want historical lookups served in-process during backtests, so that the live entry point needs no modification to be testable against history.
58. As a maintainer, I want the existing live run verified unchanged after this work, so that the feature cannot silently break the current weekly workflow.
59. As a maintainer, I want a single documented configuration file for the new behaviour, so that settings are discoverable and the existing settings files are untouched.
60. As a maintainer, I want the whole experiment runnable end to end with one command per phase, so that re-running after a data or code change is cheap.
61. As a maintainer, I want runtime to stay within a reasonable window, so that iterating on the model is not painful.
62. As a maintainer, I want a narrowing option for quicker partial runs, so that a smoke test does not require the full experiment.

## Implementation Decisions

### Scope and shape

- The work is a new, self-contained backtest package. It adds modules and tests; it modifies nothing in the existing solver, live run entry point, or settings files. This keeps upstream pull-ability intact and makes the entire feature reversible by deletion.
- The backtest package is deliberately separate from the run package, which owns the live weekly workflow.
- A JSON configuration file is introduced for the new behaviour, holding the historical data root, the test season, evaluation options, and output locations. Existing settings files are untouched.

### Data layer

- A loader normalises a community-style historical layout into a single tidy frame keyed by (season, gameweek, player). Required columns are validated up front and the loader raises listing every missing one; optional columns are detected and used opportunistically.
- The tidy frame is the single source of truth for both features and actuals. There is no separate ground-truth file.
- Schema coercion is explicit: the loader narrows types and normalises position and team naming into the solver's vocabulary at the boundary, so downstream code sees one consistent representation.
- Historical data is user-supplied and lands under a dedicated historical data root. Community CSV files are already excluded by the existing ignore rule for CSV files, so no ignore changes are needed.

### Features

- All features are within-season and lagged. The frame is grouped by (season, player) and shifted so that a row for gameweek N carries only information from strictly earlier gameweeks.
- Cross-season player identity is explicitly not attempted. Player identifiers are reassigned between seasons, so linking a player across seasons would require a name-matching step with its own error modes. Treating each season as an independent series is a deliberate simplification, not an oversight.
- Team-strength features are derived from goals for and against, themselves lagged, so no subjective fixture difficulty ratings enter the feature space.
- Optional underlying-statistic columns are accompanied by a presence indicator. When absent, values are imputed by position median rather than dropped, so the feature vector keeps a fixed width across seasons.
- The feature stage exposes one guard used directly by the leakage test: it can be asked to rebuild with all rows at or after gameweek N removed, and rows for gameweek N must be identical between the two builds.

### Model

- Three stages are fitted, because the solver needs both expected points and expected minutes, and because points are zero-inflated by non-appearance:
  - presence: probability that minutes > 0
  - conditional minutes: expected minutes given minutes > 0
  - conditional points: expected points given minutes > 0
- Expected minutes is the product of presence and conditional minutes. Expected points is the product of presence and conditional points. Availability therefore lands in expected points, which is where the solver actually respects it (expected minutes affects pool membership only, not selection).
- Estimation is a regularised linear model for the two continuous stages and a regularised logistic model for presence, both implemented in the libraries already available. No new dependency is added.
- Regularisation strength is selected by time-ordered holdout taken from within the training data only. The test season is never used for any selection.
- Features are standardised, with the scaling fitted on training data only and persisted alongside the coefficients, so that scoring later data reuses the training scaling.
- A fitted model is serialisable to disk with its coefficients, scaling, and stage metadata, so that a run can be reproduced without refitting.

### Evaluation protocol

- Evaluation is walk-forward with an expanding window. For each evaluated gameweek of the test season, the model trains on all prior seasons plus earlier gameweeks of the test season, then predicts that gameweek.
- The test season is the most recent complete season, giving a full season of evaluation points.
- A gameweek with no usable prior history is skipped and reported as skipped, so aggregates are never silently diluted.
- Projections are emitted in the schema the solver already consumes for external projection sources, one gameweek at a time. Exactly one gameweek column is emitted, because the solver aggregates expectation across every projection column present, and a single-gameweek solve would otherwise filter the player pool to empty.
- Emitted files are deliberately named so they cannot be mistaken for the external publisher's files, which the live loader auto-discovers by filename pattern.

### Layer 1 metrics

- Point accuracy: mean absolute error over the whole pool, and over players who scored.
- Ranking: per-gameweek rank correlation between projected and actual points, over the whole pool and over players who played. The played-only figure is the decision-relevant one, because selection happens among players likely to feature.
- Minutes accuracy: mean absolute error on projected minutes.
- Availability calibration: reliability bins and a Brier score for the presence probability.
- Every metric is recorded per gameweek and aggregated.

### Layer 2 solver backtest

- Each evaluated gameweek is solved from scratch as a legal squad for a single gameweek. Solving from scratch isolates projection quality from transfer and chip planning; a rolling managed squad would compound projection error with planning error and make attribution impossible.
- The real solver is invoked, not a re-implementation, so the measurement reflects shipping behaviour.
- The solve runs from a blank budget in preseason mode with a forced gameweek, a single-gameweek horizon, a single iteration, and all player-pool filtering disabled. Disabling the pool filter is required for a fair test: the default expected-minutes threshold is calibrated for multi-gameweek horizons and would exclude almost the entire pool at a one-gameweek horizon.
- Realized points come from a dedicated scoring module that applies FPL scoring rules to actual outcomes using the solver's own squad output: the starting eleven, the ordered bench, captain and vice-captain flags, and position types.
- Automatic substitution logic must respect the formation constraint — a bench player replaces a starter only when the resulting formation is legal, and the substitute goalkeeper only replaces the starting goalkeeper.
- Captaincy falls back to the vice-captain when the captain played no minutes.
- Three arms are scored through an identical realized-points path so the comparison is like-for-like:
  - model projections
  - the `ep_next` baseline
  - a hindsight-optimal squad built by projecting actual outcomes, which bounds what any projection could have achieved
- Regret is the gap between the model arm and the hindsight arm. This defines "good" relative to what was achievable in each gameweek rather than in the abstract.
- Historical team data and fixtures are served in-process during the backtest by substituting the network fetch used by the solver. Lifting that substitution out of the harness is what keeps the live entry point unmodified.

### Configuration

- The new configuration file documents historical data location, test and training seasons, model hyperparameter ranges, evaluation options, and output locations.
- A narrowing option allows a partial run over a subset of gameweeks for smoke testing.
- All output, including intermediate projections and the resolved model, is written under a single backtest output root.

### Reporting

- Tables use the tabular formatting already used elsewhere in the repo, with the same table format setting.
- A consolidated summary states the headline comparison, the per-gameweek spread, and the known limitations.
- Artefacts are written to disk so results can be reviewed without re-running.

## Testing Decisions

### What makes a good test here

Tests assert external behaviour only — observable inputs and outputs — never internal structure. The three new pure modules are exercised as functions: data in, data out, no mocking and no network. The one integration test uses the solver's existing data-preparation entry point as the boundary rather than reaching inside it, which also makes it resilient to upstream refactoring.

Tests must be able to fail. Each of the four seams below is chosen because a plausible, easy-to-make implementation bug there would otherwise produce results that look like a legitimate finding.

### Seam 1 — Leakage guard (feature construction)

The highest-value test in this project. For a given gameweek N, build features twice: once from the full frame, once from a frame with all rows at or after N removed. Every feature value for the gameweek-N rows must be identical between the two builds.

- Rationale: a feature that accidentally incorporates future information still produces a well-formed frame, a well-formed solve, and a plausible-looking metric. Nothing downstream can detect it. This is the failure that would most convincingly manufacture a fake result.
- Covers: all rolling windows, team-strength features, price momentum, and season boundaries (training on a prior season must not leak within the test season and vice versa).
- Parametrised across several gameweeks including the first gameweek of a season and a mid-season gameweek.

### Seam 2 — FPL scoring rules (realized points)

Hand-computed fixtures with asserted expected totals.

- A bench player substitutes for a non-playing starter and the formation is legal — the substitute's points count.
- A bench player would be the only possible substitute but the resulting formation is illegal — no substitution happens, and the points are not counted.
- The first-choice goalkeeper does not play, so the substitute goalkeeper comes on.
- The captain plays zero minutes, so the vice-captain's points are doubled instead.
- The captain plays, so the vice-captain is not doubled.
- All eleven starters play, so no substitution occurs and the bench contributes nothing.
- Rationale: these rules are fiddly, and an error here silently invalidates every Layer 2 number while producing no error at all.

### Seam 3 — Model plumbing (fit and predict)

Synthetic history generated from a known relationship plus noise, with the model required to beat predicting the mean on held-out synthetic data.

- Rationale: standardisation and iterative fitting are easy to get subtly wrong — fitting scaling on all data, a transposed design matrix, a divergent solve. These produce a model that trains without error and predicts poorly, which is indistinguishable from "this problem has weak signal". Without this test, a plumbing bug gets misread as a negative finding and the whole approach is wrongly abandoned.
- Also asserts that a fitted model round-trips through serialisation and produces identical predictions.

### Seam 4 — Pipeline integration (existing seam)

Emit projections for one gameweek, prepare solver data through the existing data-preparation entry point with historical lookups served in-process, and assert:

- the resulting player pool is non-empty
- every player for whom a projection was emitted is present in the prepared data
- an emptiness caused by pool filtering is distinguishable from an emptiness caused by a failed join

- Rationale: the two traps here are the pool-filter threshold emptying a single-gameweek pool, and the solver's inner merge against current-season data dropping every historical player. Both produce confusing downstream errors rather than a clear failure at the point of cause.
- This is the highest available existing seam and requires no new production seam.

### Prior art

The repository's existing tests exercise command-line option parsing through a monkeypatched argument list and stubbed collaborators, asserting the arguments reaching a stubbed boundary. The new pure-module tests are simpler than that prior art — no patching at all — and the integration test follows the same "assert at a boundary" shape.

### Verification beyond unit tests

- Loader sanity: the reconstructed per-season player set, price range, and identifier set are compared against that season's summary file before any modelling runs.
- Smoke test: a narrowed run over a few gameweeks, expecting point error in the low single digits and positive rank correlation. A wildly different value indicates a join or leakage bug rather than a modelling problem, and should be investigated as a bug first.
- Full run: whole test season; the per-gameweek table is inspected for coherent variation. A flat line across gameweeks indicates the projection is not varying, which is almost always a pipeline defect.
- Live-path regression: the existing weekly command is re-run afterwards and must reproduce the same plan as before this work.
- Linting and formatting must pass under the repository's existing configuration.

## Out of Scope

- **Gradient boosting or any additional machine-learning dependency.** A regularised linear model with good lagged features is the correct first instrument. If it cannot beat a recent-form average, additional model capacity will not rescue it, and knowing that cheaply is the point.
- **Weekly automatic retraining and live datasource registration.** The model is not wired into the live weekly run. It becomes a selectable projection source only after the backtest shows it is worth deploying.
- **Cross-season player identity linking.** Would require name matching with its own error modes; each season is treated as an independent series instead.
- **Comparison against the external spreadsheet publisher.** Its available files cover future gameweeks with no overlap with completed ones, and its contents are not archived historically, so a backtest comparison is not possible. That comparison can only be run forward from now on, and this spec does not attempt it.
- **Chip strategy, transfers, and multi-gameweek planning evaluation.** The backtest deliberately isolates projection quality by solving each gameweek from scratch.
- **Any modification to the solver, the live run entry point, or the existing settings files.**
- **A user interface or dashboard.** Output is tables and artefacts.

## Further Notes

### The baseline is the naive floor

The `ep_next` baseline is reconstructed, not archived. It is defined as the mean of actual points over the previous few gameweeks, which is the same quantity the official game's form figure is built from — and a spot check during this work found the official expected-points-next figure identical to that form figure for the large majority of players sampled. This has two consequences: the reconstruction is well-justified, and the baseline doubles as the naive floor. There is no separate weaker baseline to add. If genuine archived snapshots become available later, an adapter can substitute them without changing the harness.

### Limitations that must be printed with results

These are stated here so they cannot be quietly dropped from the report:

1. The baseline is a reconstruction, not archived values, for the reason above.
2. The external spreadsheet publisher is not in the comparison, and cannot be.
3. Historical setup is reconstructed approximately: prices, positions, and teams come from the historical files, while structural rules such as formation requirements are taken from the current season. The structural rules have not changed and are not expected to, but this is an approximation.
4. Solving from scratch is not the same as managing a real team. It measures projection quality, not transfer decision quality.
5. One season is one season. A full season is far more informative than a handful of gameweeks, but a single season carries real variance. A positive result should be confirmed on a second held-out season before it is believed.
6. If a regularised linear model with good lagged features cannot beat the recent-form baseline, more model capacity will not fix it.

### Design rationale worth preserving

- **Why the solver is not modified:** it is a clone of an actively maintained upstream project. Local modifications would make pulling upstream changes painful, and there is no reason to modify it — everything needed is already configurable, and the network lookups can be substituted in-process by the harness.
- **Why solving is done from scratch per gameweek:** a rolling managed squad compounds projection error with transfer and planning error, so a poor result could not be attributed to either. Solving from scratch answers the question actually being asked.
- **Why availability is folded into expected points rather than expected minutes:** the solver uses expected minutes only for pool membership and never to limit selection. Encoding availability anywhere else would be ignored.
- **Why the test season is the most recent complete one:** the live current season has only a few completed gameweeks, which is too few evaluation points to distinguish a real improvement from noise.

---

## Reconnaissance findings (added after dataset inspection)

These were discovered when the historical dataset was actually fetched, and they amend the sections above. Where they conflict, these win.

### The dataset schema is not what the plan assumed

The community dataset does **not** use the `merged_gw.csv` + `players_raw.csv` layout originally assumed for every season. Actual layout per season directory:

- `gws/merged_gw.csv` — the per-gameweek frame (all seasons)
- `players_raw.csv` — season-end player attributes, including price, position and team
- `teams.csv` — team attributes. **Absent for 2018-19** (404 upstream)
- `fixtures.csv` — fixture list

### `xP` is real archived expected points, and it exists for some gameweeks

This corrects the earlier claim that archived snapshots do not exist. Two forms exist:

- An `xP` column inside the per-gameweek frame
- Per-gameweek snapshot files with an `id,xP` shape

Coverage is sparse and varies by season. Archived snapshot files per season:

| Season | Snapshots | | Season | Snapshots |
| --- | --- | --- | --- | --- |
| 2018-19 | 0 | | 2023-24 | 37 |
| 2019-20 | 0 | | 2024-25 | 35 |
| 2020-21 | 38 | | 2025-26 (test) | 11 |
| 2021-22 | 38 | | 2026-27 | 4 |
| 2022-23 | 36 | | | |

For the test season the `xP` column is populated only in GWs 1-6, 8, 9, 24, 29 and 38; it is zero in every other gameweek. Snapshot files and the embedded column were verified identical on the overlap (376 players at GW9, zero mismatches), so the snapshots are canonical for provenance but add no coverage the embedded column lacks. Note the training seasons have far better snapshot coverage than the test season, which is worth remembering when interpreting per-gameweek baseline quality.

**Consequence:** the baseline arm should use archived values where they exist and reconstruct elsewhere. This is now a measured choice rather than an assumption.

### The reconstruction is measurably worse than the archived values

On the 374 players where both are available at GW9 of the test season:

| Baseline source | Mean absolute error |
| --- | --- |
| Archived `xP` | 1.498 |
| Reconstructed from recent scoring | 1.954 |

The reconstruction is roughly 30% worse. **Consequence:** the baseline arm prefers archived values, falls back to reconstruction only where none exist, and the report states reconstruction quality against the archived overlap rather than leaving it unquantified.

### Double gameweeks exist and must be aggregated

The test season contains double gameweeks in which a player appears more than once in the same gameweek: GW26 (79 players), GW33 (248) and GW36 (82), for 419 duplicated player-gameweek keys in total.

**Consequence:** the loader's "one row per player per gameweek" requirement means aggregating multiple fixtures *within* a gameweek — summing points, minutes and other additive statistics, and taking a single representative value for attributes that do not add. Dropping either row, or emitting both, would silently corrupt both the actuals and the features.

### Old seasons have a reduced schema and one needs a different encoding

| Seasons | Missing | Note |
| --- | --- | --- |
| 2018-19 | `position`, `team`, and the `teams.csv` file | file is **latin-1**, not UTF-8 |
| 2019-20 | `position`, `team` | UTF-8 |
| 2020-21 onward | none of these | UTF-8 |

Underlying statistics (`expected_goals`, `expected_assists`) begin in 2022-23. `xP` in the per-gameweek frame appears from 2020-21.

**Consequence:** the loader needs (a) an encoding fallback, and (b) a documented fallback for `position` and `team` on the two oldest seasons, drawn from that season's player attributes file. This must not be conflated with the "fail loudly on missing required columns" requirement, which applies to genuinely required columns.
