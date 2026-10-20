---
name: rf-credit-defaulter-case-study
description: Orchestrator for the Random Forest credit card defaulter classification case study (Datasets/RF_Credit_card_defaulter_dataset/Credit Card Default Data.csv, notebook at "Random_Forest_case_study/credit_Card_defaulter_rf_case_study.ipynb"). Use whenever asked to build, extend, re-run, or reason about this case study end to end. Sequences the generalized classification-* skills in order.
---

# Random Forest Credit Card Defaulter Case Study: Orchestrator

Before doing anything else, read `Datasets/RF_Credit_card_defaulter_dataset/Knowledge_Base.md`
in full. It contains the verified schema, the full data dictionary (this
dataset ships with no separate description file, so the dictionary there is
the authoritative reference), and all 11 resolved decisions — there is no
need to re-explore the raw CSV or re-derive those decisions.

## Working conventions

- The deliverable notebook lives at
  `Random_Forest_case_study/credit_Card_defaulter_rf_case_study.ipynb`.
- The Gemini key lives in `Random_Forest_case_study/.env` as
  `GOOGLE_API_KEY` (reused from the other case studies in this project).
- Every phase skill below is generalized (reusable for any future
  classification case study) and assumes the Knowledge Base has already been
  read — it does not repeat dataset-specific facts, only this dataset's
  resolved decisions do.
- If a new data quirk or decision is discovered mid-analysis, update
  `Knowledge_Base.md` rather than only fixing it inline.

## Pipeline (execute in this order)

1. **`classification-baseline-models`** — drop `ID`, one stratified
   60/20/20 split (reused everywhere after this), then fit Logistic
   Regression, KNN, Decision Tree, and Random Forest on the untouched data
   with zero EDA/scaling/encoding/imbalance handling.
2. **`classification-model-evaluation`** (applied to the baseline fits) —
   accuracy, classification report, ROC-AUC, overfit/underfit check for all
   four baseline models, one comparison table + chart.
3. **`classification-eda-profiling`** — full EDA from scratch: nulls,
   duplicates, distributions, class-imbalance profiling, with `.head()`/
   `.shape` checkpoints throughout.
4. **`classification-data-cleaning`** — fold `EDUCATION`/`MARRIAGE`
   undocumented codes into `"Other"`; no rows are dropped (0 nulls, 0
   duplicates already verified).
5. **`classification-visualization`** — the full seaborn chart suite:
   class-balance bar chart, distributions, correlation heatmap (expect the
   `BILL_AMT1..6` collinearity trap), target-split boxplots, outlier
   screening.
6. **`classification-encoding-scaling`** — numeric/categorical split
   (`PAY_*` stays ordinal/numeric, `SEX`/`EDUCATION`/`MARRIAGE` one-hot),
   VIF remediation on the `BILL_AMT*` columns, one shared
   `ColumnTransformer`/`Pipeline` with scaling for all four models.
7. **`classification-imbalance-handling`** — SMOTE (train-only) for
   Logistic/KNN, `class_weight="balanced"` for DT/RF; build both untreated
   and treated variants of each, with before/after `value_counts()`.
8. **`classification-hyperparameter-tuning`** — GridSearchCV for all four
   models with the full parameter grids (including Gini vs entropy for the
   tree models and the complete Random Forest grid), `f1_macro` scoring, cv
   inside the training split only.
9. **`classification-model-evaluation`** (applied again, at every stage
   above) — the running comparison table/chart accumulates baseline →
   EDA-no-imbalance → EDA+imbalance-handled → tuned, for all four models,
   with OOB (RF), K value (KNN), ROC-AUC at every stage, and the
   overfit/underfit verdict per row.
10. **`classification-gemini-narration`** — not a pipeline stage; the
    shared helper every other skill calls for its markdown remark cells.
    Set it up early (right after baseline models, before the first remark
    is needed) so every later phase can use it, with the logger-suppression
    fix included from the start.

Each phase's markdown cells (header, motive, remark) follow the shared
notebook conventions in the Knowledge Base — don't restate those
conventions inside each phase skill, just follow them.
