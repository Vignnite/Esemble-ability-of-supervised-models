# RF Credit Card Defaulter Case Study: Knowledge Base

Context-compaction doc. Read this instead of re-exploring the raw CSV from
scratch — it captures everything already verified about this dataset.

## File map

| Path | Purpose |
|---|---|
| `Datasets/RF_Credit_card_defaulter_dataset/Credit Card Default Data.csv` | Raw data (see schema below) |
| `Datasets/RF_Credit_card_defaulter_dataset/Knowledge_Base.md` | This file |
| `Random_Forest_case_study/credit_Card_defaulter_rf_case_study.ipynb` | The working analysis notebook (target deliverable) |
| `Random_Forest_case_study/.env` | Holds `GOOGLE_API_KEY`, reused from the other case studies in this project. Never print, log, or hardcode this value. Gitignored. |
| `Random_Forest_case_study/.env.example` | Placeholder template, safe to share |

## Dataset schema (as verified, not just as documented)

`Credit Card Default Data.csv`: **30,000 rows, 25 columns, 0 nulls in any
column, 0 fully-duplicated rows.** Every column is already numeric-coded (no
raw string categoricals), which is what makes a genuinely zero-preprocessing
baseline feasible without crashing.

No separate data-dictionary file ships with this dataset. The column meanings
below are the well-known UCI "Default of Credit Card Clients" documentation
(Taiwan, 2005), stated here as verified domain knowledge, not fabricated.

| Column | Meaning |
|---|---|
| `ID` | Row identifier, 1..30000, all unique. Not a feature — dropped before even the baseline. |
| `LIMIT_BAL` | Amount of given credit (NT dollar), continuous. |
| `SEX` | `1` = male, `2` = female. |
| `EDUCATION` | Documented: `1` graduate school, `2` university, `3` high school, `4` others. **Observed but undocumented**: `0`, `5`, `6` also appear in the data — a known quirk of this dataset, not a data-entry error unique to this copy. |
| `MARRIAGE` | Documented: `1` married, `2` single, `3` others. **Observed but undocumented**: `0` also appears. |
| `AGE` | In years. |
| `PAY_0, PAY_2, PAY_3, PAY_4, PAY_5, PAY_6` | Repayment status for September (`PAY_0`), August (`PAY_2`), July (`PAY_3`), June (`PAY_4`), May (`PAY_5`), April (`PAY_6`) 2005. **Note the naming quirk**: there is no `PAY_1` — the most recent month is named `PAY_0`, a known oddity of this dataset, not a missing column. Values: `-2` = no consumption, `-1` = paid in full/duly, `0` = revolving credit used but not yet due, `1`-`8`+ = payment delayed by that many months. This is a genuinely **ordinal** column (more negative/zero = better repayment behavior, higher = more delinquent) — encode it as ordinal/numeric, not one-hot. |
| `BILL_AMT1..BILL_AMT6` | Bill statement amount for each of the same six months, same ordering as the `PAY_*` columns. Expected to be **highly collinear with each other** (consecutive months of the same underlying balance) — the direct classification-case analogue of the CO2 case study's fuel-consumption VIF trap. |
| `PAY_AMT1..PAY_AMT6` | Amount actually paid in each of the same six months. |
| `defaulted` | **Target.** `1` = defaulted on payment next month, `0` = did not. Class split: **77.88% / 22.12%** (23,364 / 6,636) — moderately imbalanced (~3.5:1), not extreme, but enough to materially affect distance/gradient-based models if left untreated. |

## Decisions resolved with the user (apply consistently, don't re-derive)

1. **Four model families run through every stage of the notebook**: Logistic
   Regression, KNN, Decision Tree, Random Forest — not just Random Forest,
   and not just as a one-time baseline reference.
2. **`ID` is dropped before even the baseline** — a row identifier isn't a
   modeling decision the way skipping EDA/scaling/encoding/imbalance
   handling is; it's recognized as a structural non-feature from the start.
3. **One stratified 60/20/20 train/validation/test split**, done once,
   reused for the baseline and every later phase — not a rotating K-fold
   like the CO2 case study. Every comparison across stages is on the exact
   same rows.
4. **Imbalance handling by model family**: SMOTE (`imblearn`, train-split
   only) for Logistic Regression and KNN; `class_weight="balanced"` for
   Decision Tree and Random Forest (no resampling needed there). Build both
   an "imbalance acknowledged but untreated" and an "imbalance treated"
   variant of every applicable model for direct comparison.
5. **GridSearchCV for every model's hyperparameter tuning**, cv inside the
   training split only, scored on `f1_macro`. Full parameter grids,
   including the complete Random Forest grid (`n_estimators`, `max_depth`,
   `min_samples_split`, `min_samples_leaf`, `ccp_alpha`, `criterion`), not
   `ccp_alpha` alone. Decision Tree and Random Forest grids both try
   `criterion` ∈ {`gini`, `entropy`} and report which won plus how close the
   runner-up was.
6. **Overfit/underfit gate**: >10% relative gap between train and test
   accuracy (the classification analogue of the CO2 case study's 7.5% R²
   gap rule), checked at every single stage, not only after final tuning.
7. **One shared `ColumnTransformer`/`Pipeline`**, reused by all four models
   from the EDA stage onward — one-hot for the cleaned `SEX`/`EDUCATION`/
   `MARRIAGE` categoricals, `StandardScaler` for all numeric columns
   (including the ordinal `PAY_*` codes). Scaling doesn't hurt the tree
   models, so one pipeline covers everyone rather than branching per family.
8. **`EDUCATION`/`MARRIAGE` undocumented codes** (`0`, `5`, `6` for
   `EDUCATION`; `0` for `MARRIAGE`) are folded into an explicit `"Other"`
   category during cleaning, not silently kept as their own numeric codes or
   dropped as rows.
9. **`.head()`/`.shape` checkpoints after every transformation step**
   throughout EDA, plus explicit `value_counts()` on the target immediately
   before and after every SMOTE `fit_resample` call, so every claimed
   transformation has visible before/after evidence in the notebook.
10. **ROC curve + AUC tracked at every comparison stage** (baseline,
    EDA+imbalance-handled, tuned), not only for the final model. OOB score
    reported for Random Forest wherever `bootstrap=True`/`oob_score=True`.
    Winning `n_neighbors` (K) reported explicitly for KNN.
11. **Gemini key reused** from the other case studies in this project
    (`Random_Forest_case_study/.env`), with the `google_genai.types` logger
    fix applied from the very first build (see Decision/Knowledge Base note
    in `classification-gemini-narration`), not patched in later.

## Notebook conventions (same as the other case studies in this project)

- Structure per concept: markdown header → markdown "motive" cell → code
  cell → (for substantive analytical steps) a Gemini-generated markdown
  remark cell.
- No inline `#` comments in code cells.
- Visualizations: seaborn only.
- All repeated logic lives in named, reusable functions — no copy-pasted
  blocks. The preprocessing + model steps are wrapped in one
  `sklearn.Pipeline`, built once and reused across every stage.

## Build status

Not yet built as of this writing. Build via the `rf-credit-defaulter-case-study`
orchestrator skill, which sequences the generalized `classification-*` phase
skills in order.
