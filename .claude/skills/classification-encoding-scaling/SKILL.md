---
name: classification-encoding-scaling
description: Generalized phase skill for any classification case study — numeric/categorical split, one-hot vs ordinal encoding rule, VIF-based multicollinearity workflow, and feature scaling guidance (which model families need it and why one shared pipeline covers both). Use after visualization and before imbalance handling/baseline comparison.
---

# Classification Case Study: Encoding, Multicollinearity & Scaling

## Guardrails

### Numeric / categorical split and encoding

- Split columns into numeric and categorical lists explicitly, as two named
  lists reused by the shared `ColumnTransformer`.
- **Encoding rule**: a categorical column with no natural order gets one-hot
  encoding; a column with genuine order gets ordinal/numeric treatment
  (either left as-is if already numerically coded in a meaningful order, or
  `OrdinalEncoder` with an explicit category order if it's text). State
  explicitly, per column, which rule applied and why — don't one-hot a
  column just because it happens to be categorical in dtype if its values
  already carry a meaningful order (e.g. a repayment-delay-severity code or
  a satisfaction-level scale).
- **Target is never one-hot encoded** for binary classification — it's
  already (or gets mapped to) `{0, 1}`.

### Multicollinearity (VIF)

- Compute VIF for every numeric feature (`statsmodels.stats.outliers_influence.variance_inflation_factor`,
  with an intercept/constant column).
- Apply the standard risk table: 1–5 safe, 5–10 warning (investigate
  pairwise correlation before acting), above 10 critical (act immediately).
- Remedy in sequence when critical: (a) drop the single highest-VIF column,
  recompute the full VIF table, repeat until everything clears; (b) if both
  variables are needed, engineer a combined ratio/difference instead of
  dropping either; (c) only as a last resort, switch the algorithmic remedy
  (e.g. a penalized linear model or dimensionality reduction) instead of
  continuing to chase VIF.
- Expect repeated-period numeric columns (e.g. several months of the same
  underlying measurement) to be the first flagged — show the VIF table at
  each iteration, not just the final one.

### Feature scaling

- **Which model families need it**: distance-based and gradient-based
  models (KNN, logistic/linear models, SVM) are scale-sensitive — features
  on wildly different numeric ranges will distort distances or dominate the
  loss gradient. Tree-based models (Decision Tree, Random Forest, boosted
  trees) are scale-invariant — they split on thresholds per feature
  independently of magnitude.
- **One shared `ColumnTransformer`/`Pipeline` still covers all model
  families**: scaling numeric features doesn't hurt tree-based models (a
  monotonic transform doesn't change their splits), so a single
  preprocessing pipeline reused across every model in the case study is
  simpler and more production-grade than branching scaled vs unscaled
  pipelines per model family. State this reasoning explicitly in the
  notebook rather than silently scaling everything.
- Fit the scaler only on the training split (inside the pipeline), never on
  validation/test — the same leakage discipline as any other fitted
  transform.

### Junk category cleanup

- If `classification-data-cleaning` already folded undocumented codes into
  an explicit `"Other"`/`"Unknown"` category, encode that like any other
  category here — don't re-discover or re-litigate the cleanup in this
  phase.
