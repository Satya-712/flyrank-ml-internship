# Refresh Opportunity Scoring for Search Content Prioritization

## Abstract

This study develops a machine learning approach for prioritizing content items for refresh review using historical search and engagement signals. The model uses impressions, clicks, click-through rate, average search position, pageviews, and sessions from an earlier observation period to identify content associated with subsequent relative CTR decline. A Logistic Regression model was trained using March 2026 features and an April 2026 outcome window, then evaluated on a time-aware May–June 2026 test period. On the final leakage-safe test, the model achieved a ROC-AUC of 0.5837 compared with 0.4524 for a simple CTR baseline, with an improvement of 0.1313 ROC-AUC. The resulting ranking is intended as a decision-support tool for prioritizing human review and does not establish that refreshing content causes improved search performance.

---

## 1. Introduction and Problem Statement

Search-driven content portfolios contain many pages that may require periodic review, but reviewing every page manually is inefficient. This project investigates whether historical search visibility and engagement signals can be used to prioritize content items for refresh review.

The research question is:

**Which content items should be prioritized for refresh review based on their historical search visibility, CTR, engagement, and search-position signals?**

The project frames refresh opportunity as an observed relative decline in click-through rate between two consecutive monthly periods. A Logistic Regression model is used to learn this pattern from March 2026 features and April 2026 outcomes, and its ability to generalize to a later May–June 2026 period is evaluated using a time-aware test.

The objective is not to prove that content refreshes cause better Google rankings or traffic. Instead, the model provides a prioritization signal that can support human review and content-maintenance decisions.

---

## 2. Data

The analysis uses the FlyRank Internship — Pseudonymized Warehouse Release v20260703, frozen on 2026-07-03. The release contains pseudonymized client, content, daily performance, and query-level tables.

The main table used in this study is `fact_content_daily_performance`, which contains daily observations at the report-date, client, and content level. The release contains 78,835,655 rows in this table and is partitioned by month.

For model development, March 2026 was used as the feature period and April 2026 as the subsequent outcome period. For time-aware evaluation, May 2026 was used as the feature period and June 2026 as the subsequent outcome period.

Only content items with Google Search Console data available and sufficient impressions in both periods were eligible for the CTR-based outcome. The label required at least 100 impressions in both months and a positive CTR in the earlier month.

The development dataset contained 60,410 content items. The final May–June time-aware test dataset contained 64,793 content items.

The analysis used the following historical features:

- Search impressions
- Search clicks
- Click-through rate (CTR)
- Average search position
- GA4 pageviews
- GA4 sessions

The data are pseudonymized. Client names, domains, URLs, private queries, credentials, and raw warehouse exports are not included in the public research output.

---

## 3. Methodology

### 3.1 Outcome Definition

A refresh opportunity was defined as a relative CTR decline of 50% or more between the feature month and the following month.

For each eligible content item:

**Relative CTR change = (following-month CTR - feature-month CTR) / feature-month CTR**

The label was set to:

- `1` — refresh opportunity when relative CTR change <= -50%
- `0` — otherwise

Eligibility required at least 100 Google Search Console impressions in both months and a positive CTR in the feature month.

This definition describes an observed performance change. It does not imply that the content became stale or that refreshing the content would cause the subsequent CTR to improve.

### 3.2 Model

A Logistic Regression classifier was trained using the March 2026 feature period and April 2026 outcome period.

The model features were:

1. March search impressions
2. March search clicks
3. March CTR
4. March average search position
5. March GA4 pageviews
6. March GA4 sessions

Missing GA4 pageviews and sessions were imputed using medians calculated from the training data. The final time-aware test used the same training-derived medians to avoid using information from the test period.

### 3.3 Development Validation

The development dataset was divided using a stratified train/test split. This split was used for model development and initial evaluation only.

The development ROC-AUC was **0.6861**.

### 3.4 Time-Aware Validation

For the final evaluation, the model was trained using March features and April outcomes and evaluated on May features with June outcomes.

This time-aware design tests whether the learned relationship generalizes to a later period without using future outcome information during training.

### 3.5 Baseline

The baseline ranks content using inverse May CTR, so lower historical CTR receives a higher baseline score.

The final model was compared with this baseline on the same May–June test set using ROC-AUC.

### 3.6 Leakage Checks

Feature construction was restricted to information available during the relevant feature period. Future-month outcome variables and the refresh-opportunity label were not included as model features.

A final feature-level leakage check returned **PASS**.

---

## 4. Results

### 4.1 Development Performance

The Logistic Regression model achieved a ROC-AUC of **0.6861** on the stratified development test split.

The development result was used for model development and was not treated as the final generalization result.

### 4.2 Time-Aware Test Performance

On the leakage-safe May–June 2026 time-aware test:

| Metric | Final Model |
|---|---:|
| Precision | 0.3126 |
| Recall | 0.8626 |
| F1 Score | 0.4589 |
| ROC-AUC | 0.5837 |

The final model was compared with the CTR baseline on the same May–June test set.

| Method | ROC-AUC |
|---|---:|
| CTR Baseline | 0.4524 |
| Scaled Logistic Regression | 0.5837 |
| Difference | +0.1313 |

The model therefore produced a higher ROC-AUC than the simple CTR baseline on the held-out time-aware period. This result indicates that the combination of historical search and engagement features provided additional ranking signal beyond the baseline used in this experiment.

However, the final ROC-AUC of **0.5837** indicates that the predictive signal was modest. The result should therefore be interpreted as a prioritization signal rather than a highly accurate prediction of future CTR decline.

### 4.3 Ranking Output

The final model was used to generate a ranked review queue containing **64,793 eligible content items**. Each item was assigned a relative decision score and priority rank.

The ranking is intended to help focus human review on content items with stronger model signals. The scores are ranking scores rather than calibrated probabilities of future CTR decline.

---

## 5. Limitations

This study has several limitations.

1. The refresh-opportunity label is based on observed relative CTR decline and does not measure whether a content refresh actually caused a later improvement.

2. The label threshold of a 50% relative CTR decline is an analytical choice. Different thresholds could produce different class distributions and model performance.

3. Some positive outcomes include complete or near-complete CTR declines, which may reflect zero or very low clicks in the subsequent month. This can make the outcome sensitive to small click counts.

4. The final time-aware ROC-AUC of 0.5837 indicates that predictive signal is modest. The model should therefore be used for prioritization and human review rather than treated as a definitive predictor.

5. The data contain uneven availability across clients and dates, particularly for Google Search Console and GA4 signals. This may affect which content items are eligible for analysis.

6. The study covers specific monthly periods from 2026 and may not generalize to other time periods, sites, industries, or search environments.

7. The model does not establish causality between content refreshes and search performance. External factors such as query demand, competition, seasonality, technical changes, and other content changes may affect CTR and search outcomes.

8. The ranked output is a decision-support queue. Human review is required before any content change or refresh action is taken.

---

## 6. Ranked Recommendations

The final ranking should be used as a review queue rather than as an automatic content-update decision.

### Priority 1 — High model score and meaningful search visibility

Review content items that receive high model decision scores while also having substantial search impressions. These items represent cases where the model identifies a stronger refresh-opportunity signal and the content has meaningful search visibility.

### Priority 2 — High model score with low historical CTR

Review high-ranked items with comparatively low historical CTR. The combination of search visibility and weak click capture can identify pages where additional investigation may be useful.

### Priority 3 — High model score with weaker search position

Review high-ranked items with comparatively weaker average search position. These pages should be examined together with their search intent, content relevance, and competitive context before deciding whether a refresh is appropriate.

### Recommended Review Workflow

1. Start with the highest-ranked items in the model-generated queue.
2. Confirm that the page is still relevant to its intended search intent.
3. Check whether the title and content accurately match the query intent.
4. Review content freshness, completeness, and factual accuracy.
5. Check for technical or indexing issues before changing the content.
6. Record the reason for any refresh decision.
7. Measure subsequent performance separately rather than assuming that a refresh caused an improvement.

The ranking provides prioritization evidence, not a causal recommendation that every high-ranked page should be refreshed.

---

## 7. Reproducibility

The analysis was developed in Google Colab using Python, DuckDB, pandas, and scikit-learn. The warehouse data were accessed through the authorized Hugging Face dataset release using a protected access token.

The development period was March 2026 with April 2026 as the outcome window. The final time-aware evaluation used May 2026 features and June 2026 outcomes.

The final model is a scaled Logistic Regression classifier. Missing GA4 pageviews and sessions were imputed using training-period medians. The final test evaluation used the same training-derived medians to avoid test-period information leakage.

The notebook contains the feature construction, label definition, model training, validation, leakage checks, evaluation metrics, visualizations, and final ranking generation.

### Data Credit

**Data source:** FlyRank Internship — Pseudonymized Warehouse Release v20260703, frozen 2026-07-03.

The public research output does not expose client names, domains, URLs, private search queries, credentials, or raw warehouse exports.

---

## 8. Acknowledgments

This work was completed as part of the FlyRank Machine Learning Internship capstone.

The analysis uses the authorized FlyRank pseudonymized warehouse release provided for the internship. All conclusions in this study are limited to the analyzed data, methodology, and evaluation periods.

---

## Reproducibility Link

**GitHub Repository:**  
https://github.com/Satya-712/flyrank-ml-internship

**Capstone Notebook:**  
`work/notebooks/capstone_refresh_scoring.ipynb`
