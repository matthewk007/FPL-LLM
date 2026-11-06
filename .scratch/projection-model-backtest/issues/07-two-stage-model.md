# 07: Two-stage projection model

**What to build:** A trained model that predicts, for every player and gameweek, both an expected-points figure and an expected-minutes figure. It is fitted in three stages, because points are dominated by non-appearances and because the two outputs mean different things: a presence model estimating the probability a player features at all, then conditional models for minutes and for points given that the player features. Expected points and expected minutes are each the product of presence and the corresponding conditional prediction.

Availability therefore lands in expected points, which is where the solver actually respects it — the solver uses expected minutes only for player-pool membership and never to limit selection.

Estimation is regularised linear and logistic regression, implemented with libraries already present, so no new dependency is introduced. Regularisation strength is chosen by time-ordered holdout taken from within the training data only; the held-out season is never used for any selection. Feature scaling is fitted on training data and persisted with the coefficients, so scoring later data reuses the training scaling. A fitted model serialises to disk, so a run can be reproduced without refitting.

**Blocked by:** 06: Feature construction with the leakage guard

**Status:** ready-for-agent

- [ ] A fitted model predicts expected points and expected minutes for arbitrary players and gameweeks
- [ ] The three stages are separately inspectable, so a poor projection can be traced to availability, minutes, or performance
- [ ] Expected points and expected minutes are each consistent with the presence probability, so an unlikely starter projects low even at a high per-appearance rate
- [ ] The model beats predicting the mean on synthetic data generated from a known relationship plus noise
- [ ] Regularisation strength is selected from data, using only training data, and the test season is never used for selection
- [ ] A fitted model round-trips through serialisation and produces identical predictions afterwards
- [ ] No new dependency is added to the project
- [ ] Predictions are produced for the full player pool, including players with no prior history, without error
