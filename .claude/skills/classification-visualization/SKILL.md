---
name: classification-visualization
description: Generalized phase skill for any classification case study — the seaborn visualization suite (distributions, correlation heatmap, outlier boxplots, target-vs-feature comparisons, explicit class-balance chart), each with a narrated takeaway. Use after cleaning and before encoding/scaling.
---

# Classification Case Study: Visualization

## Guardrails

- **seaborn only** for chart types (matplotlib `pyplot` is fine only for
  figure/axes plumbing).
- **Every chart earns a markdown takeaway** — a plot with no interpretation
  afterward is incomplete.
- **Required chart set** (build all, combined multi-panel where there are
  several of the same kind, not scattered one-per-cell):
  1. **Class balance**: a bar chart of the target's value counts, annotated
     with the percentage split. This is not optional even if EDA profiling
     already printed the numbers — a visual of the imbalance is what
     motivates the imbalance-handling phase later.
  2. Numeric feature distributions (histogram + KDE, combined panel).
  3. Correlation heatmap across numeric features plus the target (point-
     biserial correlation is fine for a binary target) — the first visual
     warning of any multicollinearity trap (e.g. several columns measuring
     near-identical repeated-period signals, like consecutive months of the
     same underlying quantity).
  4. Boxplot/violin of each numeric feature split by target class — the
     classification analogue of a regression case's "feature vs target"
     scatter, showing whether a feature's distribution actually differs
     between classes.
  5. Outlier screening boxplots (combined panel) across numeric features,
     feeding into the cleaning/outlier-handling decision.
- **No modeling, encoding, or imbalance decisions here** — this phase
  visualizes only; VIF, encoding, and imbalance handling live in their own
  skills.
- Follow the project's shared notebook conventions (header → motive → code
  → optional Gemini remark).
