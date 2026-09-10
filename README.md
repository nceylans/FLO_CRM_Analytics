# FLO CRM Analytics

Customer segmentation, churn prediction, and Customer Lifetime Value (CLTV) analysis on FLO's omnichannel retail transaction data — 19,945 customers, analyzed through RFM scoring, K-means clustering, gradient boosting, and BG-NBD/Gamma-Gamma probabilistic models.

## Overview

FLO is an omnichannel shoe retailer; the dataset covers customers who made their last purchase in 2020–2021 through both online and offline channels. The project works through four stages — EDA, segmentation, churn prediction, and CLTV forecasting — to answer a single business question: **which customers should the CRM team prioritize, and why?**

Key results:
- **19,945 customers** analyzed, spanning first purchases from 2013 to 2021
- **10 RFM segments** (Champions, Loyal Customers, At Risk, Hibernating, etc.) cross-validated against a **4-cluster K-means segmentation**
- **32.6% churn rate** (180+ days since last purchase), predicted with a Gradient Boosting model reaching **0.72 AUC**
- **157.77 TL average 6-month CLTV** per customer via BG-NBD + Gamma-Gamma modeling, split into Bronze/Silver/Gold/Platinum tiers
- A final **priority score** (60% CLTV + 40% churn risk) flags **599 "Critical" customers** — high-value customers at real risk of leaving

## Project Structure

```
FLO_CRM_Analytics/
├── dashboard.html          # Self-contained interactive dashboard (Chart.js)
├── notebooks/
│   ├── week1_eda.ipynb     # Data cleaning, feature engineering, cohort analysis
│   ├── week2_rfm.ipynb     # RFM scoring + K-means clustering
│   ├── week3_churn.ipynb   # Churn labeling & classification models
│   └── week4_cltv.ipynb    # BG-NBD / Gamma-Gamma CLTV + priority scoring
├── outputs/                # Exported PNG charts from each notebook
└── data/                   # Raw & generated CSVs (gitignored, see Data below)
```

## Analysis Workflow

### Week 1 — EDA & Feature Engineering
Loads the raw 19,945-row dataset (`flo_data_20k.csv`), converts date columns, caps outliers at the 1st–99th percentile, and engineers core features: `total_order`, `total_price`, `avg_order_value`, `online_ratio`, and `customer_lifetime_days`. Category interest flags are derived from `interested_in_categories_12` (Kadın, Erkek, Çocuk, Aktif Spor, Aktif Çocuk). A cohort analysis by first-purchase year shows churn and average spend trending across acquisition cohorts (2013–2021).

### Week 2 — RFM & K-Means Segmentation
Computes Recency, Frequency, and Monetary scores (quintile-based, 1–5) and maps them to 10 named segments (Champions, Loyal Customers, Potential Loyalists, At Risk, Cannot Lose Them, Hibernating, About to Sleep, Need Attention, New Customers, Promising). Separately, K-means (K=4, chosen via elbow + silhouette analysis) clusters customers into Champions / Loyal Customers / Potential Loyalists / At Risk, then cross-tabulates against the RFM segments to check agreement between the two methods.

### Week 3 — Churn Prediction
Defines churn as **180+ days of inactivity** (32.6% of customers). Trains and compares Logistic Regression, Random Forest, and Gradient Boosting on RFM + behavioral features; Gradient Boosting wins with **0.72 AUC**. `customer_lifetime_days` and `num_categories` are the strongest predictors. Each customer gets a churn probability and a Low/Medium/High risk tier.

### Week 4 — CLTV & Priority Scoring
Fits a **BG-NBD model** (expected future purchases) and a **Gamma-Gamma model** (expected order value) using the `lifetimes` library to estimate 6-month CLTV per customer, then splits customers into Bronze/Silver/Gold/Platinum value tiers. Combines CLTV and churn probability into a weighted **priority score** (60% CLTV, 40% churn risk) to rank customers — surfacing 599 "Critical" customers who are both high-value and at high risk of churning.

## Dashboard

`dashboard.html` is a self-contained, dark-themed dashboard (Chart.js via CDN) with four tabs — Segmentation, Churn Risk, CLTV, and CRM Strategy — summarizing the KPIs and findings above. Open it directly in a browser; no server or build step needed.

## Data

The `data/` folder is excluded from version control via `.gitignore`. The source dataset is on Kaggle:

[FLO Data — Kaggle](https://www.kaggle.com/datasets/mustafaoz158/flo-data)

Download it and place `flo_data_20k.csv` in `data/`; running the notebooks in order (Week 1 → Week 4) will regenerate the intermediate files as it goes:
- `flo_data_20k.csv` — raw transaction data (input)
- `flo_processed.csv` — cleaned data with engineered features (Week 1 output)
- `flo_rfm.csv` — RFM scores, RFM segments, and K-means segments (Week 2 output)
- `flo_final.csv` — churn labels, churn probability, and risk tier (Week 3 output)
- `flo_cltv.csv` — BG-NBD/Gamma-Gamma CLTV estimates and value tier (Week 4 output)
- `flo_master.csv` — final merged dataset with all segments, scores, and priority tiers (Week 4 output)

## Tech Stack

- Python — pandas, NumPy
- scikit-learn — KMeans, StandardScaler, Logistic Regression, Random Forest, Gradient Boosting
- `lifetimes` — BG-NBD and Gamma-Gamma models for CLTV
- Matplotlib / Seaborn for static charts, Chart.js for the dashboard
- Jupyter Notebook

## How to Run

1. Clone the repository and download the dataset from Kaggle (link above) into `data/flo_data_20k.csv`.
2. Install dependencies:
```
   pip install pandas numpy scikit-learn matplotlib seaborn jupyter lifetimes
```
3. Run the notebooks in order — each stage reads the previous stage's output:
   `week1_eda.ipynb` → `week2_rfm.ipynb` → `week3_churn.ipynb` → `week4_cltv.ipynb`
4. Open `dashboard.html` in a browser to view the summarized results.
