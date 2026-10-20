---
name: classification-baseline-models
description: Generalized phase skill for any classification case study — how to build a deliberately "zero preprocessing" baseline across Logistic Regression, KNN, Decision Tree, and Random Forest for later comparison, and what's legitimately okay to skip vs what's a structural non-feature. Use as the very first modeling phase, before any EDA.
---

# Classification Case Study: Baseline Models (No EDA)

The point of this phase is pedagogical: show what naive modeling looks like
before any cleaning, scaling, encoding, or imbalance handling, so later
phases have a real before/after to compare against — not to produce a good
model.

## Guardrails

- **What's genuinely skipped here**: missing-value handling, duplicate
  removal, distribution/correlation analysis, outlier treatment,
  multicollinearity/VIF, feature scaling, categorical encoding, and
  imbalance handling. None of these happen before the baseline fit.
- **What's still removed first, and why it's not an EDA step**: a pure row
  identifier column (e.g. an `id`/`sno`-style column) is dropped before even
  the baseline. This isn't an analytical decision like the ones being
  skipped — it's recognizing that a row counter isn't a feature at all, the
  same way you wouldn't feed a literal index into any model. State this
  distinction explicitly in the notebook so the "zero preprocessing" framing
  stays honest rather than silently including a prep step under another
  name.
- **One stratified split, used everywhere.** Do the train/validation/test
  split once, on the target, before building any model — reuse the exact
  same row-level split for the baseline and every later phase, so
  comparisons are apples-to-apples on identical rows, with only the feature
  representation differing between phases.
- **Build all four model families on the untouched data**: Logistic
  Regression, KNN, Decision Tree, Random Forest — each fit directly on
  whatever numeric-coded columns exist, with default hyperparameters, no
  class-imbalance correction.
- **Evaluate every baseline model the same way** every later stage will be
  evaluated (see `classification-model-evaluation`): accuracy, full
  classification report, ROC-AUC, and the overfit/underfit check — so the
  baseline table and every later comparison table share identical columns
  and can be stacked together at the end.
- **Narrate what's likely to go wrong, grounded in the actual numbers**, not
  a generic disclaimer: which models are scale-sensitive and may underperform
  on unscaled mixed-magnitude columns, whether the imbalance visibly skews
  any model toward predicting only the majority class, and whether any model
  already shows a large train/test gap even before tuning.
