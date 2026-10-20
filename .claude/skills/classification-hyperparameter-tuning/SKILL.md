---
name: classification-hyperparameter-tuning
description: Generalized phase skill for any classification case study — GridSearchCV guardrails per model family (Logistic, KNN, Decision Tree, Random Forest), the Gini-vs-entropy impurity comparison, scoring choice under imbalance, and the overfit/underfit check applied after every tuned fit. Use after the imbalance-handled model comparison.
---

# Classification Case Study: Hyperparameter Tuning

## Guardrails

- **GridSearchCV, cross-validated only inside the training split.** The
  validation and test splits are never used to select hyperparameters —
  `cv` folds are carved out of the training data only; the validation split
  is at most used afterward to confirm the chosen configuration generalizes,
  and the test split is reserved for the one-time final report.
- **Scoring metric: `f1_macro`** by default under class imbalance — it
  weights both classes equally regardless of their size, so the search
  doesn't implicitly reward a model for favoring the majority class the way
  plain accuracy would. State this choice explicitly in the notebook as an
  adjustable decision, not a silent default — if the business genuinely
  prioritizes catching the minority class, `recall` or a class-weighted `f1`
  would be the deliberate alternative, and that tradeoff is worth one
  sentence of justification each time.
- **Per-model-family parameter grids**:
  - **Logistic Regression**: `C` (regularization strength, log-spaced grid),
    `penalty` (`l1`/`l2`, with a solver that supports the chosen penalty).
  - **KNN**: `n_neighbors` (the "K value" — report the winning K explicitly,
    it's the headline hyperparameter for this model), `weights`
    (`uniform`/`distance`), `metric` (e.g. `minkowski`/`euclidean`).
  - **Decision Tree**: `criterion` (`gini` vs `entropy`), `ccp_alpha`
    (cost-complexity pruning strength), `max_depth`, `min_samples_split`,
    `min_samples_leaf`.
  - **Random Forest**: everything the Decision Tree grid has, plus
    `n_estimators` — all five matter for a forest, not just `ccp_alpha`
    alone, since the ensemble's bias-variance tradeoff is shaped by tree
    count and depth together, not pruning strength in isolation.
- **Gini vs entropy — show the difference, don't just assert one**: run both
  as grid candidates for every tree-based model rather than picking one by
  convention. Gini impurity measures the probability of misclassifying a
  randomly chosen element if it were labeled according to the node's class
  distribution; entropy measures information-theoretic disorder
  (expected bits of information needed to specify the class). Gini is
  cheaper to compute (no logarithm) and tends to isolate the most frequent
  class in its own branch slightly faster; entropy is more sensitive to
  changes in class probability near the extremes and can produce slightly
  more balanced splits. In practice the two rarely disagree by much on the
  final metric — report whichever the grid search actually picked and
  whether the runner-up was close, rather than assuming they'd be identical
  without checking.
- **Overfit/underfit check after every tuned fit**: compare train vs.
  test (or train vs. validation, consistently) on the primary metric; a
  relative gap beyond the project's stated threshold (e.g. 10% for a
  classification case study, matching the regression case study's 7.5% R²
  gap rule in spirit) is flagged as `OVERFIT` (train ≫ test) or `UNDERFIT`
  (rare, train ≪ test), never left implicit in a metrics table alone.
- **Report what each model's tuning actually bought**: compare the tuned
  model's metrics against its own untuned (but imbalance-handled) version
  from the prior phase, not only against the baseline from the very start —
  both deltas matter and answer different questions.
