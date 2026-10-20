---
name: classification-data-cleaning
description: Generalized phase skill for any classification case study — identifier-column pruning, duplicate handling, missing-value imputation policy, and cleaning up undocumented/junk categorical codes. Use after EDA profiling and before visualization/encoding.
---

# Classification Case Study: Data Cleaning

## Guardrails

- **Column pruning rule (generic, reusable)**: drop any pure row identifier
  or serial-number column (`id`, `sno`, `index`, and similar) — it carries no
  predictive signal and can leak an ordering artifact into a model. State
  the decision explicitly even when it's a one-liner, so the rule stays
  visible for reuse. The same high-cardinality-near-identifier test from the
  regression case applies here too: check `nunique() / len(df)` for every
  categorical/text column, not just ones literally named `id`.
- **Duplicate handling**: default to `drop_duplicates()` on full-row
  duplicates, justified with a before/after row count. If a column was
  already dropped as an identifier, note explicitly that dedup may surface
  more repeats than a raw-frame profiling pass found (two rows differing
  only by the dropped identifier become identical once it's gone) — this is
  expected, not a bug, and should be narrated rather than silently
  overwritten.
- **Imputation policy matrix** (state explicitly even when it resolves to a
  no-op): numeric columns under 5% missing → median impute; 5–30% missing →
  median impute plus a `{col}_was_missing` flag; over 30% missing → consider
  dropping the column; categorical columns with any missing → an explicit
  `"Unknown"` category rather than the mode, unless missingness is provably
  random.
- **Undocumented/junk categorical codes**: when a column's observed values
  include codes outside its documented category set (e.g. a code meant to
  be binary or a small fixed set but with stray values like `0` or
  out-of-range numbers appearing in the data), do not silently keep them as
  their own category or silently drop the rows. Default remedy: fold every
  undocumented code into a single explicit `"Other"`/`"Unknown"` category
  (justified by its row count being negligible relative to the documented
  categories), and state the before/after value counts so the decision is
  auditable.
- **No rows are dropped for being "inconvenient"** — only exact duplicates.
  Any other row-level exclusion (e.g. a genuinely impossible value) must be
  independently justified and reported with a row count, not applied as a
  blanket rule.
- Follow the project's shared notebook conventions (motive cell before code,
  no inline comments).
