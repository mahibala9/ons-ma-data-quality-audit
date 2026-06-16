# 🇬🇧 ONS UK Mergers & Acquisitions — Data Quality & Governance Audit

The ONS UK Mergers & Acquisitions time series dataset was subjected to a systematic data quality and governance audit. Five governance aspects were assessed, and the results included a scored AI readiness evaluation.

***

## Overview

From **1969 to 2026 Q1** , the Office for National Statistics releases quarterly and annual statistics on mergers and acquisitions involving UK companies. These statistics cover domestic, inward (foreign corporations purchasing UK targets), and outward (UK companies acquiring abroad) deals.

The dataset is audited in this project as it would be assessed prior to being fed into a machine learning or data engineering pipeline. It provides an answer to the query:

> **Can an automated or AI-driven analytics system use this dataset?**

***

## Governance Scorecard

| Dimension | Score | Verdict |
|---|---|---|
| Completeness | 3 / 5 | The regional sub-series is 100% absent in quarterly rows; it is intended to be annual only, but it is not documented. |
| Consistency | 3 / 5 | There is a discrepancy between the stated total and sum of components in 56 out of 213 quarterly rows (26%). |
| Timeliness | 5 / 5 | June 2026 saw an update to 2026 Q1; September 2026 is the next release date. |
| Temporal Continuity | 5 / 5 | From 1969 Q1 to 2026 Q1, there were 229 straight quarters with no pauses. |
| Schema Quality | 3 / 5 | Three inconsistent `£m` header formats over 90 columns; one column name mistake (`Ouward`) |

**Overall: 3.8 out of 5 for the core domestic series' quarterly trend analysis.
Before using AI/ML, regional sub-series and component breakdowns must be validated.**

***

## AI Readiness Assessment

| Criterion | Assessment | Ready? |
|---|---|---|
| Label availability | The dataset is unsupervised and lacks labels. Only appropriate for predicting or anomaly detection; not suitable for categorization without further labeling. | Partial |
| Feature completeness | The core quarterly series (outward/inward totals, domestic acquisitions) is finished. Regional sub-series cannot be utilized directly as quarterly features; they must first be aggregated annually. | Partial |
| Temporal leakage risk | The most recent quarter is tentative and could be revised by ONS. The most recent one or two quarters should be marked as unreliable by any automated workflow that uses this data. | ⚠️ Flag |
| Structural break risk | The 2008–09 financial crisis, the 2016 Brexit referendum, and the 2020 COVID-19 pandemic were found to be three significant structural breaches. Without windowing, a model trained over the whole period would pick up knowledge from essentially distinct market regimes. | ⚠️ Flag |
| Data volume | For traditional time-series models (ARIMA, Prophet, Exponential Smoothing), 229 quarterly rows are adequate. Insufficient without augmentation for deep learning. | ✅ Yes |

**Recommended use:** Time-series forecasting and descriptive analytics are limited to the primary quarterly domestic M&A series.

**Not recommended for:** Tasks involving classification without further labeling; cross-sectional machine learning employing regional sub-series without first performing an annual aggregation.

***

## Key Findings

### 1. Completeness — regional series are structurally sparse
The 118 columns in the dataset, which are divided by geography (EU, USA, Europe, Americas, Asia, Africa, and Oceania), cover inbound, outward, and domestic transactions. The regional sub-series are reported **annually only** and show up as `NaN` in all quarterly rows, although the core domestic and outward/inward totals are well-populated in quarterly rows. The file itself does not contain any documentation about this.

### 2. Consistency — component series do not reconcile in 26% of quarters
The ONS reported domestic acquisition total (`M&A: Domestic: Number of companies acquired`) was compared to the total of two component series (M&A of independents + sales of subsidiaries between groupings) in an internal cross-check. 56 of the 213 similar rows, including severe outliers in the pre-1987 period where component series seem to be unreported, exhibit a difference larger than 2.

### 3. Temporal continuity — backbone is complete ✅
There are no missing quarters in the quarterly series, which runs continuously from **1969 Q1 to 2026 Q1** (229 expected, 229 found). For any time-series use case, this is a huge benefit.

### 4. Schema quality — naming inconsistencies
- **Typo:** `Asia Ouward Disposals Value :£m` (missing `t` in "Outward")
- **Inconsistent £m suffixes:** three formats used across 90 columns:
  - ` :£m` — 26 columns
  - `: £m` — 33 columns
  - ` : £m` — 31 columns

***

## Visualisations

### UK Domestic M&A — Quarterly Trend 1969–2026
The quarterly time series of domestic acquisitions reveals three distinct
economic eras:
- **Late 1980s boom** — peak activity approaching 460 acquisitions per quarter
- **Post-2008 crash** — sharp sustained decline to below 100 per quarter by 2013
- **Post-COVID recovery** — gradual rebound through 2021–2024

### Internal Consistency — Mismatching Quarters
The bulk of quarters are located near the 45° reference line in the scatter plot of reported total vs. reconstructed total, with a noticeable cluster of outliers above the line focused in the pre-1987 period where component series are zero-filled rather than accurately reported.

***

## Repository Structure

```
ons-ma-data-quality-audit/
├── ons-ma-data-quality-audit.ipynb   # Main notebook
├── outputs/
│   ├── ma_trend.png                  # Quarterly trend chart
│   ├── consistency_scatter.png       # Internal consistency scatter
│   └── governance_scorecard.csv      # Scored findings table
└── README.md
```

***

## Dataset

| Field | Detail |
|---|---|
| Source | Office for National Statistics (ONS) |
| Series | Mergers & Acquisitions involving UK Companies |
| File | `am.csv` |
| Shape | 292 rows × 118 columns |
| Coverage | 1969 Annual → 2026 Q1 |
| Granularities | Annual (57 rows) + Quarterly (229 rows) |
| Release date | 2 June 2026 |
| Next release | 1 September 2026 |
| URL | https://www.ons.gov.uk/businessindustryandtrade/changestobusiness/mergersandacquisitions |

***

## Governance Actions Before AI Use

1. Remove from quarterly feature sets any columns marked `: Annual`.
2. To prevent cross-regime leaking, use a rolling train or test window
3. In any automatic output, mark the most recent quarter as preliminary.
4. Standardize column names to eliminate typos and inconsistent £m spacing.

***

## Tech Stack

- **Python 3.12** — data processing and analysis
- **Pandas** — data loading, cleaning, reshaping, audit logic
- **Matplotlib** — trend charts and scatter plots
- **Kaggle Notebooks** — execution environment

***

## Author

**mahibala9** · June 2026  
Kaggle: [ons-ma-data-quality-audit](https://www.kaggle.com/code/mahathisatyawada/ons-uk-m-a-data-quality-governance-audit)

***

*Data © Office for National Statistics, licensed under the Open Government Licence v3.0*
