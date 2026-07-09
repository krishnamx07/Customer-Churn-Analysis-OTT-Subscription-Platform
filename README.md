# Customer Churn Analysis - OTT Subscription Platform

End-to-end churn analytics project built on a relational SQLite database, using Python (pandas, numpy) for cleaning and feature engineering, and matplotlib/seaborn for visualization. Identifies which customers are churning, why, and where the revenue exposure is concentrated.

## Overview

Subscription businesses lose far more from churn than they save by skipping retention work. This project takes a 3-table relational dataset (customer demographics, subscription/contract details, and support interactions) and turns it into a churn risk model with 20+ KPIs, segment-level breakdowns, and clear, data-backed recommendations.

**Key result:** Monthly-contract subscribers churn at **55.6%** vs **8.3%** for annual contracts - a 6.7x gap that is the single strongest churn driver in the dataset.

## Dataset

Source: `customer_churn.db` (SQLite), 3 relational tables joined on `customerid`:

| Table | Fields |
|---|---|
| `db_customer` | customerid, name, country, state, gender, dob |
| `db_subscription` | customerid, subscription_start_date, plan_type, contract_type, cancellation_date, monthly_charges, cltv, churn_score |
| `db_support` | customerid, complaint_date, escalations, csat_score |

## Tech Stack

- **Database:** SQLite
- **Data extraction & manipulation:** Python, pandas, numpy, sqlite3
- **Visualization:** matplotlib, seaborn
- **Environment:** Jupyter Notebook

## Project Workflow

1. **Data extraction** : connect to SQLite, pull all 3 tables into pandas via `sqlite3` + `pandas.read_sql`.
2. **Data cleaning** : type correction, column renaming, dropping unused columns, missing-value handling.
3. **Relational join with de-duplication** : the support table has a one-to-many relationship with customers (some customers filed multiple complaints). Deduplicating to one row per customer *before* merging was essential to avoid row fan-out (a left join against the raw table inflated 21 customers into 23 rows).
4. **Feature engineering** : churn flag, customer tenure (days), complaint count, and a 3-tier churn risk bucket (Low / Medium / High) derived from a continuous churn score.
5. **Exploratory analysis** : churn rate by plan type, contract type, and state; ARPU; revenue at risk; escalation rate; escalation–churn correlation.
6. **Visualization** : trend, comparison, and correlation charts via matplotlib/seaborn, including a correctly-encoded correlation heatmap (ordinal encoding for ordered categorical fields, not arbitrary label encoding).
7. **Insights & recommendations** : translated into a one-page executive summary (see `/report`).

## Key Insights

- **Churn rate:** 28.6% | **Retention:** 71.4%
- **Contract type is the dominant churn driver** : monthly (55.6%) vs annual (8.3%) churn rate
- **Basic plan has the highest churn (60%)** but the lowest revenue exposure per customer
- **Escalations correlate with churn** (r = 0.47) : support friction is a leading indicator
- **Revenue at risk:** Rs 73.94/mo (18.7% of total MRR), Rs 2,047 in CLTV
- Cancellations cluster around September : worth cross-checking against pricing/service changes

Full write-up with charts: [`report/churn_analysis_report.pdf`](./report/churn_analysis_report.pdf)

## Repository Structure

```
├── churn_analysis.ipynb           # main analysis notebook
├── customer_churn.db              # source SQLite database
├── exported_churn_data.csv        # cleaned, merged dataset (post-pipeline)
├── report/
│   └── churn_analysis_report.pdf  # executive insights report
└── README.md
```

## Running Locally

```bash
git clone https://github.com/krishnamx07/customer-churn-analysis.git
cd customer-churn-analysis
pip install pandas numpy matplotlib seaborn jupyter
jupyter notebook churn_analysis.ipynb
```

## Notes on Data Quality

A deliberate part of this project was catching and documenting real data-pipeline bugs rather than hiding them:
- A left-join row fan-out caused by an undeduplicated one-to-many support table (21 → 23 rows) - fixed by deduplicating before merging.
- A correlation matrix silently computed on a 5-row slice instead of the full dataset due to a stray `.head()` - fixed and re-validated.
- A type-mismatch bug where an already-encoded numeric column was compared against a string, silently zeroing out real values - fixed by removing the redundant re-encoding step.

These are documented as a reminder that **"Restart Kernel & Run All" before calling a notebook done** is not optional.

