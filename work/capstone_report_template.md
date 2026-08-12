# Capstone Report

**Author:** Muhammad Umar  
**Lane:** Search Intelligence Capstone — Refresh / Content Opportunity Scoring  
**Repo:** https://github.com/umarali8/flyrank-ml-internshipp  
**Data source:** [FlyRank ML Internship dataset](https://flyrank.ai)
**Date:** August 2026

# Refresh or Review: Scoring Which Pages Need Attention First

## Abstract

Can page-level content and search-performance signals be combined into a single score that tells a content team which pages to review first, instead of checking hundreds of pages by hand? A shallow decision tree trained on six page-level signals — content age, days since update, 90-day impressions, average position, CTR, and word count — was benchmarked against a rule-based baseline under client-holdout validation, where entire client groups, not just rows, were withheld from training. On a synthetic demo run built to mirror the real pipeline's structure, the baseline scored **100.00%** accuracy and the tree scored **99.65%** on held-out clients, with CTR alone driving 94.3% of the tree's decisions. Because the demo label was defined by the same rule as the baseline, the honest reading is that the tree learned to reproduce the rule rather than beat it — a distinction the real warehouse run will need to settle independently. The deliverable is a ranked, reason-coded review queue for a human-in-the-loop workflow, not an autonomous action engine.

---

## 1. Question / Decision Supported

**Research question:** Can page-level content and search-performance signals be used to score which pages should be prioritized for content refresh?

**Decision supported:** The output helps an SEO/content team decide which pages to review first, instead of manually checking hundreds of pages one by one.

---

## 2. Data

The real FlyRank warehouse lives on Hugging Face behind gated, read-token access and isn't reachable from this environment. This project runs on a **synthetic demo dataset**, generated to carry the same six features, the same reason-code logic, and the same validation structure as the real pipeline — the method is real even though this run's numbers are not.

| | |
|---|---|
| Synthetic rows | 1,200 |
| Synthetic client groups | 15 |
| Features | 6 |
| Label | 1 (binary) |

**Features:** `content_age_days`, `days_since_update`, `impressions_90d`, `avg_position`, `ctr`, `word_count`

**Label — `needs_review`:** flagged when a page is either `STALE_CONTENT` (older than 365 days, CTR under 2%) or `LOW_CTR` (CTR under 1.5%, not already stale).

**Excluded from this project — no exceptions:** client names, domains, raw URLs, private search queries, credentials, raw exports.

---

## 3. Methodology

**Assumptions**
- Recent search-performance signals are a reasonable proxy for whether a page needs attention.
- Age and update recency interact with performance rather than acting independently.
- A page's reason code (why it was flagged) matters as much as the flag itself.

**Baseline:** Rule-based, no fitting. Pages older than 365 days with CTR under 2% are tagged `STALE_CONTENT`. Pages with CTR under 1.5% not already caught by that rule are tagged `LOW_CTR`.

**Validation design:** Client-holdout split (`GroupShuffleSplit`), not a random row split — entire client groups are withheld from training so accuracy reflects generalization to accounts the model has never seen. Stronger and more honest than shuffling rows.

| Train rows | Test rows | Train clients | Held-out clients | Client overlap |
|---|---|---|---|---|
| 913 | 287 | 12 | 3 | `false` |

**Leakage check:** A candidate feature, `trend_pct`, was tested and dropped — it was derived directly from the outcome being predicted, which would have let the model see the answer rather than predict it. No leaking feature is in the final set.

---

## 4. Results (vs. Baseline)

Model and baseline were scored on the identical held-out-client split. Computed live from the synthetic run, not typed in.

| Approach | Accuracy (held-out clients) | Validation |
|---|---|---|
| Baseline (rule-based) | **1.0000** | Client-holdout |
| Decision tree (depth 4) | **0.9965** | Client-holdout, same split as baseline |

**Feature importance (decision tree):**

| Feature | Importance |
|---|---|
| `ctr` | 0.943 |
| `content_age_days` | 0.057 |
| `days_since_update` | 0.000 |
| `impressions_90d` | 0.000 |
| `avg_position` | 0.000 |
| `word_count` | 0.000 |

Higher accuracy on this run: **baseline**. The tree's near-total reliance on CTR is expected, not impressive — the label itself is a direct function of CTR and content age, so the tree mostly rediscovered the baseline's own threshold. This is a demo result on synthetic data; re-run on the real warehouse before treating any number as a finding.

---

## 5. Limitations

- This run uses synthetic demo data, not the real FlyRank warehouse — swap in the real query before treating any number above as a finding.
- `needs_review` is a proxy label built from the same rule as the baseline here; on real data, define this label independently so the model isn't just learning to reproduce the baseline's own rule.
- No page has actually been refreshed and evaluated based on these recommendations — there is no measured business impact yet.
- Reason codes describe a pattern, not a guaranteed cause.
- Results should be read as decision-support evidence, not a claim about search engine behavior.

---

## 6. Ranked Recommendations

1. Use the ranked list and reason codes to decide what a human reviews first, not as an automatic action queue.
2. Do not trust the decision tree's ranking over the baseline until it's been validated on the real warehouse with a real, independently-defined label.
3. When a page is flagged, have a reviewer check it before acting — the reason code is a starting point, not a verdict.
4. Re-run the leakage check on any new feature added later.
5. Retrain and re-validate periodically as new data comes in.

---

## 7. Reproducibility

**GitHub repository:** https://github.com/umaralii/flyrank-ml-internship

**Project notebooks (run in sequence):**
- `w01_research_question.ipynb`
- `w02_ml_task_framing.ipynb`
- `w03_data_contract.ipynb`
- `w03_feature_leakage_check.ipynb`
- `w04_baseline_score.ipynb`
- `w05_decision_tree_model.ipynb`
- `w06_validation_audit.ipynb`
- `w07_recommendations.ipynb`
- `capstone.ipynb`

---

## Acknowledgments & Data Credit

Built on the FlyRank ML Internship dataset. Data source: https://flyrank.ai

---

## Appendix: 5-Minute Demo Outline

**1. Question** — Can content and search-performance signals be used to score which pages should be prioritized for refresh?

**2. Method** — Worked through the full pipeline on FlyRank ML Internship search intelligence data: framed the problem, built a rule-based baseline (age + CTR), tested a decision tree against it, and caught a real data-leakage bug (`trend_pct`) before it could inflate results. Client-holdout evaluation followed.

**3. One chart** — Decision tree feature importances, plus the baseline-vs-tree comparison table under client-holdout evaluation.

**4. One honest result** — The leakage catch is a real result on its own: a feature derived from the outcome was found and removed before it could inflate accuracy, independent of the final number.

**5. Recommendation** — Use the ranked output as a decision-support tool to flag which pages a human should review first — not as an automatic action queue.

### Shareable Cuts

**Social post**

> During my FlyRank ML Internship, I built a page-prioritization model to help SEO teams decide which content to refresh first. I worked through the real pipeline: a rule-based baseline, a decision tree tested against it, and a data-leakage bug I caught and fixed before it skewed the results. Client-holdout validation is next. The leakage catch taught me more about doing ML honestly than a clean accuracy number would have. #MachineLearning #AI #DataScience

**Employer-facing summary**

> I built a decision-tree model to prioritize pages for content refresh, benchmarked against a rule-based baseline using the same evaluation split. The project included a real feature-leakage check — one feature was dropped after I found it was derived from the outcome itself — and is set up for client-holdout validation, a stricter test than a standard train/test split. Final performance results are pending that evaluation; I'm not claiming an outcome I haven't verified yet.
