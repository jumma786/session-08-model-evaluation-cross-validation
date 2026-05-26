# Model Card — SLA Breach Triage Classifier

## Intended use

This model supports **early triage** of incoming service requests. It produces a risk score that flags requests at higher likelihood of breaching their SLA so triage analysts can prioritise attention before the deadline is missed. It is a decision-support tool; every flag is reviewed by a human.

## Dataset and validation strategy

Synthetic workplace service-request log: 900 rows, 19 columns, binary target `sla_breach` (42.8% positive). Thirteen features were used after excluding three post-outcome fields (`*_after_close`) that leak future information, the `request_id` identifier, and `priority_level` (a separate target). The data was split 75/25 with stratification. Models were compared using **5-fold stratified cross-validation on the training portion** with identical folds and scoring. The 25% holdout was touched once at the end as a sanity check.

## Metrics

| Metric    | Logistic Regression | Random Forest    |
|-----------|---------------------|------------------|
| Accuracy  | 0.562 ± 0.036       | **0.615 ± 0.026** |
| Precision | 0.490 ± 0.040       | **0.569 ± 0.041** |
| Recall    | **0.546 ± 0.076**   | 0.408 ± 0.069    |
| F1        | **0.515 ± 0.047**   | 0.473 ± 0.055    |
| ROC-AUC   | 0.588 ± 0.030       | **0.602 ± 0.042** |

Overfitting signal (train minus CV): Logistic Regression ~0.07–0.09 across every metric; Random Forest ~0.37–0.57. The forest's training F1 is 0.99 against a CV F1 of 0.47.

## Business interpretation

The two errors carry different costs. A **false negative** — failing to flag a request that subsequently breaches — degrades a customer outcome and may carry contractual consequences. A **false positive** costs an analyst a brief review. Missed breaches are materially worse than false alarms, so **recall is the primary metric**, with F1 as a tiebreaker to prevent a degenerate "flag everything" model.

## Limitations

- **Synthetic data**: classroom dataset only; real-world performance is unknown.
- **Modest discriminative power**: holdout ROC-AUC of 0.61 — the model adds signal but is not strong on its own.
- **Subgroup performance varies sharply by region**: holdout recall ranges from 0.59 (England) to 0.11 (Wales). Slice sizes are small but the gap is too large to ignore.
- **Leakage risk**: `updates_first_24h` is safe only if prediction happens after 24 hours; remove it if triage runs at intake.
- **No time-drift signal**: the data has no temporal ordering, so drift cannot be assessed here.

## Responsible use caveat

Before real-world use this model would require: validation on real operational data using a time-based split; a formal fairness review across region, customer type and department; a decision threshold tuned to the operational cost ratio; a human-in-the-loop process; and performance-drift monitoring. The model must never auto-prioritise, auto-close, or auto-escalate without analyst review.

## Recommendation

**Use Logistic Regression for this scenario.** It scores higher on recall (0.55 vs 0.41) and F1 (0.51 vs 0.47) under identical folds — matching the business cost asymmetry of this task — and generalises far more reliably (7-point train-to-CV F1 gap vs 51 points for the forest). The forest's accuracy edge (0.61 vs 0.56) comes from being correct more often on the easy *no-breach* majority while missing more of the *breach* minority, which is the wrong trade-off when breaches are the costly class.
