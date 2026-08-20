# Predicting Near-Term Search-Visibility Decline for Content Review Prioritization

## Abstract

This study asks whether historical search-performance signals can identify content pages that are likely to experience a meaningful decline in search impressions over the following 30 days. Using the FlyRank pseudonymized warehouse release, the analysis builds a forward-looking feature set from the preceding 90 days and defines a future decline when target-window impressions fall below 80% of the immediately preceding 30-day reference period. A transparent historical-visibility baseline is compared with Logistic Regression and Random Forest using a client-held-out evaluation and Precision@K ranking metrics. Logistic Regression achieves Precision@20 of 0.85, Precision@50 of 0.86, and Precision@100 of 0.83, compared with 0.55, 0.68, and 0.74 for the baseline. The result is directional decision-support evidence that a simple learned model can improve the prioritization of pages for human review, but it does not prove causality or guarantee that a refresh will recover performance.

## Introduction / Problem Statement

Content teams often have more pages to review than they can investigate at once. The practical decision is therefore not whether every page is healthy or unhealthy, but which pages deserve attention first.

This capstone asks:

> Can historical search-performance signals identify pages that are likely to experience a meaningful search-impression decline in the next 30 days, so reviewers can prioritize which pages to inspect first?

The intended output is a ranked review queue. A high-ranked page is a candidate for investigation, not an automatic instruction to refresh or change the page.

This project follows the **Refresh / Content Opportunity Scoring** lane.

## Data

This study uses the FlyRank pseudonymized warehouse release:

- Release: `flyrank_pseudonymized_warehouse_release_v20260703`
- Source: `central_data_warehouse`
- Daily fact coverage: 2025-01-27 through 2026-06-30
- Main table: `fact_content_daily_performance`
- Supporting tables: `dim_clients` and `dim_content`

The daily fact table is at report-date × client × content grain.

For this experiment:

- Historical feature window: 2026-03-03 to 2026-05-31
- Reference window: 2026-05-02 to 2026-05-31
- Future target window: 2026-06-01 to 2026-06-30

The final working dataset contains **148,118 eligible observations**, with a future-decline rate of **66.1%**.

The analysis excludes client names, domains, URLs, raw private queries, credentials, and raw text. Pseudonymous identifiers are used only for grouping and validation, not as model features.

## Methodology

### Decision-point and time-window design

The analysis is explicitly forward-looking. Features are calculated only from information available before the June target window.

The historical feature window covers 2026-03-03 through 2026-05-31. The 30 days immediately preceding the target month provide the reference period. The target period is 2026-06-01 through 2026-06-30.

No observations from the target window are included in the model feature matrix.

### Future decline label

A content item is labeled as a future decline when:

`future_30d_impressions < 0.80 × reference_30d_impressions`

A minimum historical impression threshold is used to reduce the influence of extremely low-volume pages.

This is a future outcome rather than the starter proxy label `is_declining_label = trend_direction == "down"`.

The fields `trend_direction` and `trend_pct` are therefore not used as model features.

### Features

The model uses historical observable signals including:

- 90-day impressions
- 90-day clicks
- average search position
- sessions
- engaged sessions
- pageviews
- scroll events
- active search days
- CTR
- engagement rate
- scrolls per pageview
- log-transformed impressions, clicks, and sessions
- search-day activity rate

Client and content identifiers are retained only for grouping and validation.

### Baseline

The transparent baseline is:

`baseline_score = log1p(impressions_90d)`

The baseline represents a simple review-priority policy in which pages with greater historical exposure are prioritized first.

### Models

Two learned models were evaluated:

- Logistic Regression
- Random Forest

The purpose was to test whether a learned ranking improves prioritization over a transparent baseline rather than to maximize model complexity.

### Validation

The baseline and learned models were evaluated on the same client-held-out test set.

Client identifiers were used only to form the grouped split, ensuring that pages from a held-out client were not also used for training.

The main ranking metrics were:

- Precision@20
- Precision@50
- Precision@100

These metrics match the practical use case of prioritizing a limited review queue.

### Leakage checks

The following fields were excluded from the feature matrix:

- `future_decline`
- `impressions_future_30d`
- `impressions_reference_30d`
- `future_change_pct`
- `client_hash_id`
- `content_hash_id`
- `trend_direction`
- `trend_pct`

The feature window ends before the future target window begins.

## Results

### Model vs baseline

| Method | Precision@20 | Precision@50 | Precision@100 | Average Precision |
|---|---:|---:|---:|---:|
| Logistic Regression | **0.85** | **0.86** | **0.83** | 0.6285 |
| Random Forest | **0.85** | 0.78 | 0.78 | **0.7032** |
| Baseline | 0.55 | 0.68 | 0.74 | 0.6387 |

Logistic Regression is the strongest method at all three top-K review cutoffs.

At Precision@50, Logistic Regression achieves **0.86**, compared with **0.68** for the baseline.

Interpreted as a 50-page review queue, this corresponds to approximately:

- 43 future-decline pages in the Logistic Regression top 50
- 34 future-decline pages in the baseline top 50

on this validation sample.

Random Forest ties Logistic Regression at Precision@20, but trails it at Precision@50 and Precision@100. Random Forest has the highest Average Precision, indicating stronger overall ranking performance across the evaluated list, while Logistic Regression is better focused at the review-queue cutoffs that matter most for this decision.

The result is measured performance on this validation design. It is not a guarantee of future traffic behavior.

### Feature interpretation

The largest Logistic Regression coefficient magnitudes in the current run were associated with `sessions_90d` and `pageviews_90d`, followed by historical scroll and search-activity measures and click/CTR variables.

These coefficients describe associations learned from standardized inputs. They are not causal effects.

## Limitations & Honest Framing

This work is **decision-support**, not causal inference.

The target identifies pages whose search impressions decrease by more than 20% during the target 30-day period relative to the immediately preceding 30-day reference period. It does not establish why a page declined or whether refreshing that page would cause recovery.

Several different processes can resemble decline, including:

- seasonality
- consolidation between related pages
- changes in search-result behavior
- ordinary traffic noise

The experiment also uses one defined future target window and one client-held-out validation design. Performance may therefore differ across other periods, clients, or traffic conditions.

The ranked queue should be used to prioritize human investigation, not as an automatic instruction to refresh, rewrite, prune, merge, or otherwise change content.

All findings are framed using observed, measured, directional, and decision-support language.

## Ranked Recommendations

The final queue ranks held-out content items by Logistic Regression probability of future decline.

The recommended action playbook is:

### High-visibility pages

**Reason code:** `high_visibility_page`

**Suggested action:** Review for refresh or protection.

Pages with high historical visibility and elevated model risk deserve earlier investigation because a meaningful decline on highly exposed content may represent a consequential review opportunity.

### High-demand position opportunities

**Reason code:** `high_demand_position_opportunity`

**Suggested action:** Review content and search-position opportunity.

Pages with substantial historical demand but weaker average position may deserve investigation into content quality, search intent alignment, and positioning.

### Visible low-CTR pages

**Reason code:** `visible_low_ctr_page`

**Suggested action:** Review title/meta and intent match.

Pages with meaningful historical visibility and low CTR may warrant review of click-capture and intent alignment.

### Deeper-position pages

**Reason code:** `deep_position_risk`

**Suggested action:** Review whether the page still merits investment.

Pages with weaker average position should be investigated before deciding whether to improve, protect, monitor, or reconsider investment.

### Top-20 review queue

The notebook generates the complete ranked top-20 queue from the held-out evaluation set.

The queue is intended to support human review. A high model probability does not guarantee that the page is declining for a particular causal reason or that a refresh is the correct intervention.

## Reproducibility

The complete analysis is implemented in:

`work/notebooks/capstone.ipynb`

The notebook:

1. Installs the required packages.
2. Authenticates to the gated FlyRank Hugging Face release without storing the token in code.
3. Queries hosted Parquet data through DuckDB.
4. Constructs the historical feature window and future target window.
5. Performs leakage checks.
6. Builds the transparent baseline.
7. Trains Logistic Regression and Random Forest.
8. Evaluates the methods with client-held-out validation and Precision@K.
9. Generates the ranked recommendation queue and paper artifacts.

The full warehouse and Hugging Face access token are not committed to the public repository.

The deployed paper URL is stored in:

`submission/paper_url.txt`

## Acknowledgments & Data Credit

Built on the **FlyRank ML Internship dataset**.

Data source: [FlyRank](https://flyrank.ai)

This paper uses the pseudonymized FlyRank internship warehouse and follows the repository's public-safety rules. No private client names, domains, raw queries, credentials, or raw content are published.
