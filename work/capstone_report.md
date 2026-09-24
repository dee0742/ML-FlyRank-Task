# Content Refresh Opportunity Ranking

- **Author:** Dino Kawade
- **Lane:** Applied Search Intelligence — Content Refresh Opportunity Ranking
- **Repo:** https://github.com/dee0742/ML-FlyRank-Task
- **Date:** 2026-09-24

## 0. Abstract

Can observable content and search-performance signals rank pages that are showing an observed decline in impressions so a reviewer can decide which pages to inspect first? This study uses a public-safe starter slice of 30,000 content rows with 44 original columns across 32 pseudonymized clients. I compare a transparent staleness-and-search-demand baseline with a Random Forest ranking model using a client-grouped 80/20 validation split and Precision@K metrics. On the held-out split, the Random Forest measured Precision@20/50/100 of 0.70/0.64/0.62, compared with 0.40/0.38/0.38 for the baseline, against a test-set decline base rate of 0.511. The output is intended as human-in-the-loop decision support for prioritizing content review, not as a causal explanation of decline or a guarantee that refreshing a page will improve performance.

## 1. Introduction / Problem

Content teams cannot manually inspect every page at the same time. The practical decision is therefore which pages should enter a limited review queue first.

This project frames that decision as a ranking problem: use observable content and search-performance signals to place pages with an observed decline label near the top of a review queue. A high model score means higher measured ranking priority on the evaluation setup; it does not mean that a refresh will cause traffic or rankings to improve.

## 2. Data

The reproducible analysis uses data/raw/content_refresh_anonymized.csv, a public-safe starter slice containing 30,000 content rows and 44 original columns. The data contains pseudonymized IDs, content properties, search context, and aggregated engagement/performance signals.

There are 32 unique pseudonymized clients. The target is is_declining_label = 1 when trend_direction == down. In this released data, 16,262 rows are labeled down, giving an overall decline rate of 0.5421.

The target source fields trend_direction and trend_pct are excluded from model features. client_id and content_id are also excluded as model features; client_id is used only for grouped validation.

## 3. Methodology

The baseline is a transparent review-priority score combining percentile-ranked content staleness (days_since_last_update) and search demand (search_volume) with equal weights.

The learned model is a Random Forest classifier with 300 trees, random_state=42, class_weight=balanced_subsample, and min_samples_leaf=2. Numeric features are median-imputed and standardized; categorical features are imputed and one-hot encoded.

Validation uses an 80/20 GroupShuffleSplit grouped by client_id, with random_state=42. The resulting split contains 23,837 training rows and 6,163 test rows with zero client overlap. The main ranking metrics are Precision@20, Precision@50, and Precision@100 because the operational task is prioritization of a limited review queue.

## 4. Results

On the same client-held-out test set:

| Method | Precision@20 | Precision@50 | Precision@100 |
|---|---:|---:|---:|
| Random Forest | 0.70 | 0.64 | 0.62 |
| Staleness + demand baseline | 0.40 | 0.38 | 0.38 |

The test-set decline base rate was 0.511. At Precision@50, the Random Forest measured 0.64 while the baseline measured 0.38.

These are measured results for this dataset and validation setup. They should not be interpreted as a production guarantee, causal effect, or estimate of traffic recovered after a refresh.

## 5. Limitations & Honest Framing

The target is an observed trend rule rather than independently verified ground truth. The analysis therefore identifies items associated with the observed decline label rather than proving why an item declined.

The analysis evaluates the 30,000-row starter slice rather than claiming to model the full internship warehouse. A single client-grouped split also does not capture every possible time or cohort shift.

Precision@K measures ranking quality for review prioritization. It does not measure traffic recovered, business impact, ROI, or whether a content refresh would improve performance.

The model should therefore remain a decision-support signal. A human reviewer should decide whether a page should be refreshed, diagnosed, or monitored.

## 6. Ranked Recommendations

1. **Review high model-risk pages first.** Use the learned ranking to allocate limited review time.
2. **Give additional attention to high-demand pages showing decline.** The action playbook combines model/ranking signals with visible demand and freshness signals.
3. **Use reason codes and failure cases during review.** A high score is a prioritization signal, not a final action.
4. **Do not automate content changes from the score alone.** Keep refresh, redirect, deletion, or rewrite decisions with a human reviewer.
5. **Validate on future unseen releases before operational use.** The current measurements are directional evidence from one public-safe evaluation setup.

## 7. Reproducibility

The main reproducible notebook is work/notebooks/capstone.ipynb.

- Dataset: data/raw/content_refresh_anonymized.csv
- Split: GroupShuffleSplit, 80/20, random_state=42, grouped by client_id
- Model: RandomForestClassifier, 300 trees, random_state=42, min_samples_leaf=2
- Ranking metrics: Precision@20, Precision@50, Precision@100
- Leakage exclusions: target, trend-derived fields, and identifiers
- Action playbook: work/notebooks/w07_action_playbook.ipynb

The notebooks contain the code that recalculates the reported metrics and produces the evaluation charts and reviewer queue.

## 8. Acknowledgments & Data Credit

Built on the FlyRank ML Internship dataset and the public-safe anonymized starter release provided for the internship. Data source: https://flyrank.ai

## 9. Case-study framing

This case study applies the research workflow to a concrete content-review problem: a search/content team has more pages to inspect than it can review immediately, so it needs a transparent way to prioritize pages showing an observed decline.

The analysis combines a simple baseline, a learned ranking model, grouped validation, leakage checks, and a reviewer-facing action playbook. The measured result is useful as a directional comparison of ranking approaches on the released evaluation setup; it is not evidence that any particular content intervention will cause a search-performance improvement.