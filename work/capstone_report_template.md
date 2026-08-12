Capstone Report
Author: Muhammad Umar
Lane: Refresh / Content Opportunity Scoring
Repo: https://github.com/umarali8/flyrank-ml-internship
Date: August 2026



## 1. Problem framing

This project supports the decision of prioritizing webpages that may benefit from content refresh.

Unit of analysis: one webpage in one time window.

Output: a ranked list of pages with a priority score and a reason code (e.g. `STALE_CONTENT`, `LOW_CTR`).

Human action: a content team reviews the ranked pages and reason codes, and decides which pages are actually worth updating.

Cost of a wrong call: time wasted refreshing pages that didn't need it, or real opportunities missed because they weren't flagged.

Machine learning helps here because hundreds of pages can't be manually reviewed one by one — the model surfaces a starting point for review, not a final answer.

## 2. Data safety

Real source: the FlyRank ML Internship warehouse (Hugging Face, gated). This notebook currently runs on a synthetic stand-in dataset with the same six features and reason-code logic, clearly labeled as synthetic in the notebook itself — swap this for the real DuckDB-over-`hf://` query from starter notebook 03 before treating anything here as a finding.

Features used: content age, days since last update, 90-day impressions, average search position, CTR, word count.

Excluded data: client names, domains, raw URLs, private search queries, credentials, and raw exports.

Leakage checks:
- A feature called `trend_pct` was tested and dropped — it was directly derived from the outcome being predicted, so keeping it would have let the model "see the answer" instead of predicting it honestly.
- No future-window or label-derived fields were used as features.
- The synthetic client IDs used for the holdout split are used only for grouping, never as a predictive feature.

## 3. Baseline

The baseline is a rule-based score: pages older than 365 days with CTR under 2% are tagged `STALE_CONTENT`; pages with CTR under 1.5% not already caught by that rule are tagged `LOW_CTR`.

The baseline is applied directly (no fitting) and evaluated on the exact same client-holdout split as the Decision Tree, so the comparison is fair.

## 4. Model / analysis

A Decision Tree was chosen over a more powerful model (e.g. random forest or gradient boosting) deliberately — the goal wasn't the highest possible score, it was making sure a reviewer could see which signals actually drove a flag. A decision tree keeps that reasoning readable.

Features used: content age, days since last update, 90-day impressions, average search position, CTR, word count.

Features deliberately excluded: `trend_pct` (leakage), client identifiers, any future-window signal.

Target definition: whether a page needs review (`needs_review`), based on the same age/CTR pattern as the baseline rule in this synthetic run — this is a known limitation, noted in Section 6.

## 5. Evaluation

Validation design: client-holdout — entire clients are held out of training, not just random rows, so the test checks generalization to clients the model has never seen. This is a stricter test than a random row-level split.

On the synthetic demo run: 913 training rows / 287 test rows, 12 training clients / 3 held-out test clients, verified zero client overlap between train and test.

Results on the synthetic run: baseline accuracy 1.000, Decision Tree accuracy 0.9965, both measured on the same held-out clients. **This result is not meaningful as a real finding** — the synthetic label was built from the same rule as the baseline, so the baseline was always going to score close to perfect on it. [PENDING: real accuracy comparison once the real warehouse data and an independently-defined label are used.]

Feature importance (Decision Tree, synthetic run): CTR dominated (0.943), followed by content age (0.057); the remaining four features had ~0 importance in this run. [PENDING: re-check on real data — this ranking may not hold once the label isn't derived from the baseline rule.]

## 6. Interpretation

The most useful result from this phase isn't the accuracy number — it's the leakage catch. Finding and removing `trend_pct` before it inflated results is a real, verifiable outcome of the process, independent of final model performance.

Known limitation to flag honestly: the synthetic run's label was built using the same rule as the baseline, so the near-identical accuracy scores mostly confirm the model can learn a rule it was implicitly given, not that it beats the baseline on a genuinely independent target. On real data, the label needs to be defined independently of the baseline rule, or this comparison isn't meaningful.

Results here are directional and decision-support only — they do not prove causation, and they say nothing about how any search engine actually ranks pages.

## 7. Recommendation

Recommended actions:
- Use the ranked list and reason codes to decide what a human reviews first — not as an automatic action queue.
- Do not treat the Decision Tree's ranking as more trustworthy than the baseline until it has been validated on real data with an independently-defined label.
- When a page is flagged, have a reviewer check the actual page before acting — the reason code is a starting point, not a verdict.
- Re-run the leakage check on any new feature added later — this is a real, recurring risk, not a one-time fix.
- Retrain and re-validate periodically as new data comes in.

Confidence and limitations: no page has been refreshed and evaluated based on this system's recommendations yet — there is no measured business impact to report. Human review is required before any content changes are made based on this output.

## 8. Reproducibility

Repository: https://github.com/umarali8/flyrank-ml-internship

Project notebooks (per assignment spec, place under `work/`):
- w01_research_question.ipynb
- w02_ml_task_framing.ipynb
- w03_data_contract.ipynb
- w03_feature_leakage_check.ipynb
- w04_baseline_score.ipynb
- w05_decision_tree_model.ipynb
- w06_validation_audit.ipynb
- w07_recommendations.ipynb
- capstone.ipynb (built from `Capstone_Refresh_Content_Scoring.ipynb`)

Environment: Python via Google Colab. Main libraries: pandas, scikit-learn, matplotlib. Fixed random seed (42) used throughout for reproducibility.

Run process: open notebooks in order; replace the synthetic data cell in the capstone notebook with the real `hf://` warehouse query; run baseline and Decision Tree training; run client-holdout evaluation; review results and export recommendations.

