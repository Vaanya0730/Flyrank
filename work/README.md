# FlyRank Capstone — Search-Visibility Decline Prioritization

## Overview

This capstone investigates whether historical search-performance signals can identify content pages that are likely to experience a meaningful decline in search impressions over the following 30 days.

The project follows the **Refresh / Content Opportunity Scoring** lane and produces a ranked review queue intended to help prioritize human investigation.

## Research Question

> Can historical search-performance signals identify pages that are likely to experience a meaningful search-impression decline in the next 30 days, so reviewers can prioritize which pages to inspect first?

## Method

The analysis uses a forward-looking design:

- Historical feature window: **2026-03-03 → 2026-05-31**
- Reference window: **2026-05-02 → 2026-05-31**
- Future target window: **2026-06-01 → 2026-06-30**

# FlyRank Page Decline Prediction

A page is labeled as a **future decline** when:

> future 30-day impressions < 80% of reference 30-day impressions

The analysis uses observable historical search and engagement signals and excludes target-derived fields and identifiers from the model feature matrix.

## Models Compared

- Transparent historical-visibility baseline
- Logistic Regression
- Random Forest

Validation uses a client-held-out split so pages from held-out clients do not appear in training.

## Results

| Method              | Precision@20 | Precision@50 | Precision@100 | Average Precision |
| -------------------- | -----------: | -----------: | -------------: | -----------------: |
| Logistic Regression | **0.85**     | **0.86**     | **0.83**       | 0.6285              |
| Random Forest        | **0.85**     | 0.78         | 0.78           | **0.7032**          |
| Baseline              | 0.55         | 0.68         | 0.74           | 0.6387              |

Logistic Regression provides the strongest top-K review-queue performance in this validation experiment.

At Precision@50, Logistic Regression achieves 0.86 compared with 0.68 for the baseline.

## Output

The project produces:

- model comparison metrics
- feature interpretation
- error analysis
- ranked recommendations
- reason codes
- review-priority recommendations
- paper-ready tables and charts

## Repository Structure

\`\`\`
work/
├── README.md
├── capstone_report.md
└── notebooks/
    └── capstone.ipynb
\`\`\`

## Reproducibility

The main analysis is implemented in:

\`\`\`
work/notebooks/capstone.ipynb
\`\`\`

The notebook queries the hosted FlyRank warehouse using DuckDB, constructs the feature and future-label windows, trains the models, evaluates the rankings, and generates the recommendation output.

The full warehouse and Hugging Face credentials are not committed to the repository.

## Public Research Paper

The deployed research paper is available at:

https://vaanya0730.github.io/Flyrank/

## Data Credit

Built on the FlyRank ML Internship dataset.

Data source:

https://flyrank.ai

## Honest Framing

This work is decision-support, not causal inference.

The ranking identifies pages that are worth reviewing based on observed historical signals. It does not prove why a page declined, predict Google's algorithm, or guarantee that refreshing a page will improve future performance.
