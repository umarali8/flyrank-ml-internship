# Capstone Report

**Author:**  Muhammad Umar

**Lane:** Search Intelligence Capstone — Refresh / Content Opportunity Scoring

**Repository:** [https://github.com/umarali8/flyrank-ml-internshipp](https://github.com/umarali8/flyrank-ml-internshipp)

**Data source:** [FlyRank ML Internship dataset](https://flyrank.ai/)

**Date:** August 2026
---

## 1. Problem framing

This project supports the decision of prioritizing webpages that may benefit from content refresh.

**Unit of analysis:** One webpage (content item) belonging to one anonymized client.

**Output:** A binary decline flag (`is_declining`) and a ranked list of pages, backed by a decision tree score.

**Human action:** SEO/content teams can review the ranked/flagged pages and decide which pages should be reviewed first, instead of manually checking hundreds of pages one by one.

**Cost of a wrong call:** Time may be wasted reviewing pages that do not need changes, or pages that genuinely need a refresh may be missed.

Machine learning helps because a large number of pages (30,000 rows across 32 clients in this dataset) cannot be manually reviewed efficiently. The model helps identify patterns in content and search-performance signals and supports faster, more consistent triage decisions.

## 2. Data safety

The project used the **FlyRank ML Internship Starter dataset** (public-safe, anonymized release) — 30,000 rows, 53 columns, 32 anonymized clients.

**Used features:**

- Search volume, competition, CPC
- Word count, character count
- Impressions (90d), clicks (90d), pageviews (90d), sessions (90d)
- Content age (days), days since last update
- CTR, average position
- Engagement rate, scroll rate

**Excluded data:**

- Client names
- Domains / raw URLs
- Private search queries
- Credentials
- Raw exports

**Leakage checks:**

- Outcome-derived fields were not used as model features.
- `is_underperformer`, `is_initial_refresh_candidate`, `is_quick_win`, `needs_ctr_fix`, `needs_engagement_fix`, `health_score`, and `ai_opportunity` were excluded as leakage.
- `trend_pct` was specifically caught and removed — it is derived from the outcome and would have inflated results if left in as a feature.
- `client_id` was only used for grouping in the holdout split, not as a predictive feature.
- No client-identifying information was included in the work folder.

## 3. Baseline

The baseline was a simple search-performance rule used as the point of comparison for the machine-learning model.

For fair evaluation, the baseline and the Decision Tree model were compared using the **same client-holdout split** — no client appeared in both training and testing.

**Baseline result:** 0.8793 accuracy on held-out clients.

The baseline acts as a reference point to measure whether the ML approach provides a more structured prioritization method than a simple rule.

## 4. Model / analysis

The project used a **Decision Tree Classifier** (`max_depth=4`, `random_state=42`) for the content refresh prioritization lane.

The Decision Tree method was selected because it provides an interpretable ML approach that can identify patterns between content/search-performance signals and page decline.

**Target definition:** `is_declining` (binary) — 24,437 negative cases, 5,563 positive cases. This target is a proxy used to rank/flag pages for refresh review, not a guarantee of future outcomes.

**Features used (15):** `search_volume`, `competition`, `cpc`, `word_count`, `char_count`, `impressions_90d`, `clicks_90d`, `pageviews_90d`, `sessions_90d`, `content_age_days`, `days_since_last_update`, `ctr`, `avg_position`, `engagement_rate`, `scroll_rate`.

**Features deliberately excluded:** `trend_pct`, `health_score`, `ai_opportunity`, `is_underperformer`, `is_initial_refresh_candidate`, `is_quick_win`, `needs_ctr_fix`, `needs_engagement_fix`, `client_id`.

The model was designed to support content prioritization decisions, not to predict search engine algorithms or guarantee ranking improvements.

## 5. Evaluation

The model was evaluated using **client-holdout validation**: entire clients were kept in either training or testing, with no client overlap.

**Evaluation approach:**

- Training rows: 23,837 (25 clients)
- Test rows: 6,163 (7 clients)
- Client overlap: 0 → **PASS**, no client leakage.
- Same split used for both the baseline and the Decision Tree model, for a fair comparison.

**Results:**

| Approach | Accuracy | Validation |
|---|---|---|
| Baseline | 0.8793 | Client-holdout |
| Decision Tree | 0.8717 | Client-holdout |

The baseline slightly outperformed the Decision Tree on this held-out split.

**Feature importance (Decision Tree):**

| Feature | Importance |
|---|---|
| impressions_90d | 81.97% |
| scroll_rate | 6.55% |
| clicks_90d | 5.78% |
| content_age_days | 5.27% |
| char_count | 0.32% |
| competition | 0.12% |
| all remaining features | 0.00% |

**Error analysis:**

- The model relies heavily on one signal (`impressions_90d`), which may make it brittle for pages with mixed or unusual performance patterns.
- It mainly depends on available search-performance features rather than deeper content-quality signals.
- Results should be interpreted as decision-support evidence rather than a precise prediction of outcomes.

## 6. Interpretation

The model identified patterns in search-performance signals that can help prioritize pages for content refresh, though it did not beat the simple baseline on this evaluation.

**Key observations:**

- `impressions_90d` dominates the model's decisions, suggesting visibility volume is the strongest available signal for flagging decline.
- `scroll_rate`, `clicks_90d`, and `content_age_days` provide secondary signal.
- Many candidate features (search volume, word count, CPC, CTR, average position, engagement rate, days since last update) carried effectively zero importance in this model — a simple rule using the top few signals may be just as effective as the tree.

**Feature interpretation:**

- Impressions show search visibility, and their strong weight suggests decline is closely tied to visibility loss.
- Scroll rate and clicks reflect user engagement with the page.
- Content age provides context on how long the page has existed without change.

The results are directional and should be used as decision-support evidence. The model does not prove causation and does not predict search engine ranking algorithms.

## 7. Recommendation

The model output supports ranked content-refresh decisions for content teams, alongside the baseline it was benchmarked against.

**Recommended actions:**

- Use the model ranking to decide which pages a human reviewer should check first.
- Compare the model ranking with the baseline before making content decisions, since the baseline currently performs at least as well.
- Review the signals behind each flagged page (especially `impressions_90d` and `scroll_rate`) before recommending a refresh.
- Check every new feature for data leakage before adding it to the model.
- Retrain and re-validate the model when new search-performance data becomes available.

**How a content team can use it:** An editor can review the ranked pages, check which signals drove the flag, and decide which updates are worth applying.

**Confidence and limitations:** The recommendations provide directional decision-support based on observed search signals. Human review is required before making content changes. The model does not guarantee improved rankings or traffic, and on this dataset it did not outperform a simple rule-based baseline.

## 8. Reproducibility

The project can be reproduced by following the notebooks in sequence from the GitHub repository.

**Repository:** https://github.com/umarali8/flyrank-ml-internshipp

**Project notebooks:**

- `w01_research_question.ipynb`
- `w02_ml_task_framing.ipynb`
- `w03_data_contract.ipynb`
- `w03_feature_leakage_check.ipynb`
- `w04_baseline_score.ipynb`
- `w05_decision_tree_model.ipynb`
- `w06_validation_audit.ipynb`
- `w07_recommendations.ipynb`
- `capstone.ipynb`

**Environment:**

- Python environment used through Google Colab.
- Main libraries: pandas, numpy, matplotlib, scikit-learn.
- Random seed: a fixed random state (`42`) was used throughout for reproducibility.

**Run process:**

1. Open notebooks in order.
2. Run data loading and inspection steps.
3. Define target and features, and run the leakage check.
4. Run client-holdout split and train the baseline and Decision Tree model.
5. Review evaluation results, feature importance chart, and exported recommendations.

## Acknowledgments & data credit

Built on the FlyRank ML Internship dataset. Data source: [https://flyrank.ai](https://flyrank.ai)

> I built a decision-tree model to prioritize pages for content refresh, benchmarked against a rule-based baseline using the same evaluation split. The project included a real feature-leakage check — one feature was dropped after I found it was derived from the outcome itself — and is set up for client-holdout validation, a stricter test than a standard train/test split. Final performance results are pending that evaluation; I'm not claiming an outcome I haven't verified yet.
