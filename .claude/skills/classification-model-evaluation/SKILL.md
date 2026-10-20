---
name: classification-model-evaluation
description: Generalized phase skill for any classification case study — the full evaluation toolkit (classification report, confusion matrix, ROC curve + AUC tracked at every stage, OOB score for bagging/Random-Forest models, K value for KNN, and the overfit/underfit gap rule). Use for every model fit from the baseline stage onward, not just at the end.
---

# Classification Case Study: Model Evaluation

## Guardrails

- **One shared `evaluate()` helper, reused for every model at every stage**
  (baseline, EDA-no-imbalance, EDA+imbalance-handled, tuned) so every
  comparison table has identical columns and nothing is computed ad hoc per
  cell. It should return at minimum: accuracy, macro-averaged precision/
  recall/f1, and ROC-AUC.
- **Full `classification_report` (precision/recall/f1 per class, plus
  macro/weighted averages)** — never report accuracy alone, especially
  under class imbalance, where accuracy can look deceptively good while the
  minority class is barely being predicted at all.
- **Confusion matrix** for at least the final/best model (and any stage
  where the result is surprising) — a plotted heatmap, not just the raw
  numbers, so false positives vs false negatives are visually obvious.
- **ROC curve + AUC tracked at every comparison stage, not only the final
  model.** Plot it (or at minimum report the AUC number) for baseline,
  imbalance-handled, and tuned versions of every applicable model, so the
  AUC trend across stages is visible in the same way the accuracy/F1 trend
  is — a model can have a stable AUC while its accuracy swings with the
  classification threshold, which is itself worth noting if it happens.
- **OOB (out-of-bag) score for Random Forest** (and any other bagging
  ensemble): report `oob_score_` whenever `bootstrap=True` and
  `oob_score=True` are set — it's a near-free validation-quality estimate
  from the samples each tree didn't see, worth reporting alongside (not
  instead of) the held-out validation/test metrics.
- **K value for KNN**: always report the winning `n_neighbors` explicitly
  next to its metrics — for KNN, this is the headline "what hyperparameter
  made the difference" the way `ccp_alpha` or `n_estimators` is for trees.
- **Overfit/underfit gate**: compute the relative gap between train and
  test (or train and validation) on the primary metric; beyond the
  project's stated threshold (e.g. >10% for a classification case study),
  flag `OVERFIT` (train ≫ test) or `UNDERFIT` (train ≪ test), else `OK`.
  Apply this at every single stage, not only after final tuning — a
  baseline model can already be badly overfit, and that's worth surfacing
  immediately rather than waiting until the end.
- **Comparison tables and charts accumulate across stages** — append each
  new stage's row to one running table (mirroring a regression case study's
  running model-comparison table), and build at least one comparative bar
  chart (e.g. accuracy/F1/AUC per model per stage, grouped) so the
  improvement path is visible at a glance, not just readable row by row.
