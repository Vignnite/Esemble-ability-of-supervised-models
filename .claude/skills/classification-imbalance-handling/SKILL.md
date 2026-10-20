---
name: classification-imbalance-handling
description: Generalized phase skill for any classification case study with an imbalanced target — the decision matrix between SMOTE (train-only) for distance/gradient-sensitive models and class_weight="balanced" for tree-based/ensemble models, and the hard rule that resampling never touches validation/test data. Use after encoding/scaling and before baseline/tuned model comparisons.
---

# Classification Case Study: Imbalance Handling

## Guardrails

- **Visualize the imbalance before treating it** (reuse the class-balance
  chart from `classification-visualization` if already built, or build it
  here) — state the minority-class percentage explicitly, since that number
  determines how aggressive the remedy needs to be.
- **Print `y_train.value_counts()` (and shape) immediately before and
  immediately after every `fit_resample` call.** The before/after row counts
  and class split are the direct evidence that SMOTE did what it claims —
  never just assert "SMOTE balanced the classes" without showing the actual
  counts changing from imbalanced to balanced (or to whatever target ratio
  was configured).
- **Decision matrix, by model family, not a blanket SMOTE-everywhere rule**:
  - **Distance/gradient-sensitive models** (Logistic Regression, KNN, SVM,
    gradient-boosted models without native class weighting): use **SMOTE**
    (`imblearn.over_sampling.SMOTE`) to synthetically oversample the
    minority class, because these models' decision boundaries are directly
    shaped by the relative density of each class in the training data, and
    skew toward the majority class without correction.
  - **Tree-based/ensemble models with native class weighting** (Decision
    Tree, Random Forest, and similar): use `class_weight="balanced"`
    instead of resampling. These models split on impurity criteria that can
    be reweighted directly per class, so synthetic oversampling is
    redundant extra complexity — reweighting the impurity calculation
    achieves the same correction without inventing synthetic rows or
    changing the training set's actual size/distribution.
- **Hard rule: resampling only ever touches the training split.** `SMOTE`
  must be fit (`fit_resample`) only on the training portion, after it has
  already been separated from validation and test — never on the full
  dataset before splitting, and never on validation/test data in either
  direction. Validation and test sets must always reflect the real-world
  class distribution, since that's what the model will actually face in
  production. State this explicitly next to every `fit_resample` call site.
- **Build both an "imbalance acknowledged but untreated" and an "imbalance
  treated" variant of each applicable model** when the case study calls for
  a before/after comparison — don't skip straight to the treated version,
  since the whole point is demonstrating the delta.
- **Evaluate with imbalance-aware metrics, not accuracy alone.** A model
  that always predicts the majority class can still score high accuracy on
  an imbalanced target; report precision/recall/f1 per class and macro-
  averaged versions (see `classification-model-evaluation`) so the
  imbalance-handling decision's actual effect is visible.
