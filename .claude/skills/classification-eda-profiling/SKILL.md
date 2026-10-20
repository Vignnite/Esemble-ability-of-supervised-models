---
name: classification-eda-profiling
description: Generalized phase skill for any classification case study — initial data loading, shape/dtype/null/duplicate profiling, numeric distributions, and class-imbalance profiling of the target. Use when starting the full-EDA phase of a classification notebook (after any deliberately-skipped no-EDA baseline), not tied to any specific dataset.
---

# Classification Case Study: EDA & Profiling

Applies to any classification dataset's full-EDA phase. If the project has a
dataset-specific orchestrator skill (e.g. one pointing at a Knowledge Base
doc), read that first for the verified schema — this skill states the
*procedure*, not any one dataset's specific numbers.

## Guardrails

- **Always compute, never assume**: `df.shape`, `df.dtypes`, `df.isnull().sum()`,
  `df.duplicated().sum()`, `df.nunique()` per column — even if a prior
  baseline stage already skipped this on purpose, the full-EDA phase must
  show its own work from scratch, not reference the baseline's numbers.
- **Class-imbalance profiling is part of EDA profiling, not a separate
  afterthought.** Report the target's class counts and proportions
  explicitly (`y.value_counts()` and `.value_counts(normalize=True)`) here,
  before any modeling decision gets made about it — the imbalance-handling
  skill depends on this number being established first.
- **Numeric distributions**: histogram + boxplot (combined multi-panel
  figure, not one chart per cell) for every numeric column, commenting on
  skew and plausible range, same precision bar as a regression case study's
  EDA — every claim backed by a printed number.
- **Categorical column profiling**: cardinality and top-frequency values for
  every categorical/coded column. Flag any column whose observed values
  don't match its documented category set (e.g. an undocumented code like
  `0` appearing in a column whose dictionary only lists `1`-`4`) — this
  feeds directly into `classification-data-cleaning`'s cleanup step, don't
  silently normalize it here.
- **No modeling or cleaning decisions in this phase.** This phase only
  observes and reports; decisions (dropping junk codes, imputation, capping)
  belong to `classification-data-cleaning` and later skills.
- **Show the data after every transformation, not just at the start.**
  Every step across the whole EDA pipeline (profiling, cleaning, encoding,
  scaling, imbalance handling) that changes the frame's shape or contents
  should be followed by a quick `df.head()` and `df.shape` (or `X.shape`)
  check — this is the running visual proof that each step did what it
  claimed, not just a printed sentence asserting it.
- Follow the project's shared notebook conventions (markdown header →
  motive → code; no inline comments; seaborn-only charts) without restating
  them here.
