# Session 08 — Model Evaluation and Cross-Validation

## Objective

Compare two classifiers — Logistic Regression and Random Forest — for predicting workplace SLA breaches, using cross-validation and metrics aligned to business risk rather than raw accuracy.

## Dataset

Synthetic workplace service-request dataset (900 rows, 19 columns). Target: `sla_breach` (binary, 42.8% positive). Data dictionary (in the original session pack) flags leakage and fairness risks.

## Models compared

- **Logistic Regression** — `class_weight='balanced'`, `max_iter=500`
- **Random Forest** — 180 trees, `max_depth=8`, `class_weight='balanced'`

Both models share identical preprocessing (`StandardScaler` for numeric features, `OneHotEncoder` for categoricals) inside an sklearn `Pipeline`. Preprocessing lives inside the pipeline so fold-specific fit happens during cross-validation — no leakage from validation folds into preprocessing statistics.

## Validation strategy

- 75/25 stratified train/holdout split
- **5-fold stratified cross-validation** on the training portion only
- Identical folds and scoring metrics for both models (fair comparison)
- Holdout used exactly once, at the end, as a final sanity check

## Key result

**Logistic Regression is the recommended model** for this triage task, despite lower accuracy.

| Metric            | Logistic Regression | Random Forest |
|-------------------|---------------------|---------------|
| Recall (primary)  | **0.546**           | 0.408         |
| F1 (secondary)    | **0.515**           | 0.473         |
| Accuracy          | 0.562               | 0.615         |
| ROC-AUC           | 0.588               | 0.602         |
| Train–CV F1 gap   | **0.08**            | 0.51          |

Recall and F1 are prioritised because missed breaches (false negatives) cost more than false alarms in this scenario. The Random Forest's accuracy advantage comes from over-predicting the majority "no breach" class. Its enormous train–CV gap (0.99 train F1 vs 0.47 CV F1) also signals severe overfitting.

## Limitations

- Synthetic data — real-world performance unknown
- Recall on the Wales subgroup is 0.11 vs 0.59 for England (small slice but a real fairness flag)
- Modest discriminative power overall (ROC-AUC ≈ 0.6)
- `updates_first_24h` is safe only if prediction happens after the first 24 hours
- No temporal ordering, so drift cannot be assessed from this data

See `reports/model_card_section.md` for the full responsible-use writeup.

## Repository structure

```
session-08-model-evaluation/
  README.md
  data/
    workplace_model_evaluation_dataset.csv
  notebooks/
    Session_08_Practical_Activity.ipynb    # executed top-to-bottom
  reports/
    model_card_section.md
    metrics_table.csv
    fairness_slice_region.csv
  visuals/
    confusion_matrix.png
    roc_curve.png
```

## How to run

```bash
cd notebooks/
jupyter notebook Session_08_Practical_Activity.ipynb
# Runs end-to-end. Regenerates everything in reports/ and visuals/.
```

Requires: `pandas`, `numpy`, `scikit-learn`, `matplotlib`, `jupyter`.

## Author

Jumma Mohammad Teli — May 2026
