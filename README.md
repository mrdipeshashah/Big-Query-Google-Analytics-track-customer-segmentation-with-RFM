# GA4 BigQuery RFM Segmentation Toolkit

A production-grade collection of BigQuery SQL models for performing Recency, Frequency, and Monetary (RFM) Customer Segmentation using Google Analytics 4 (GA4) e-commerce export data.

---

## 📌 Context & Data Maturity

This repository serves as a Level 1 Data Maturity framework for e-commerce customer segmentation:
* Horizon: Ideal for 6–12 month operational windows to identify high-value tiers, at-risk buyers, and churn candidates without needing complex external ETL pipelines.
* Server-Side Compatibility: Fully optimized for both standard web streams and Server-Side Google Tag Manager (sGTM) implementations.
* Optimized Architecture: Designed to minimize scan costs and processing overhead in BigQuery via strict partition pruning and windowed analytics.

---

## 📁 Repository File Layout

├── 01_RFM-Discrete-Rules-Option-A.sql
├── 02_RFM-Percentile-Dynamic-Option-B.sql
├── 03_Days-Between-First-And-Last-Purchase.sql
└── README.md

### 1. 01_RFM-Discrete-Rules-Option-A.sql (Fixed Lookup Model)
* Approach: Uses discrete integer scoring (1 to 5) and a standard 11-segment lookup mapping.
* Best Used For: Matching traditional categorical RFM frameworks, standardized BI dashboards, and step-by-step reporting walkthroughs.
* Key Logic: Concatenates rfm_recency, rfm_frequency, and rfm_monetary into a 3-character string (e.g., '555', '511') and maps it against an explicit lookup CASE statement.

### 2. 02_RFM-Percentile-Dynamic-Option-B.sql (Production Model)
* Approach: Continuous mathematical percentiles (PERCENT_RANK()) combined with composite threshold scoring.
* Best Used For: Scaled production pipelines and stores with non-standard order distributions (e.g., high AOV low-frequency stores, or subscription models).
* Key Logic: Replaces static lookup strings with fluid threshold rules (rfm_recency >= 4 AND rfm_frequency >= 4), ensuring 100% of customers fall into active behavioral segments without edge-case NULL values.

### 3. 03_Days-Between-First-And-Last-Purchase.sql (Lifecycle Auxiliary)
* Approach: Summary query computing customer lifespan metrics.
* Key Metrics: Orders count, total spend, AOV, first purchase date, most recent purchase date, and total active days elapsed between orders.

---

## ⚡ SQL Optimization & Performance Highlights

All models in this repository incorporate key performance and data-quality optimizations:

1. User Identity Stitching: Utilizes COALESCE(user_id, user_pseudo_id) to unify guest browsing cookies with authenticated account logins, preventing duplicate single-purchase profiles.
2. Partition Pruning: Filters _TABLE_SUFFIX >= FORMAT_DATE('%Y%m%d', DATE_SUB(CURRENT_DATE(), INTERVAL 24 MONTH)) to limit BigQuery byte-scans to active evaluation windows.
3. Single-Pass Aggregations: Derives dataset-wide reference dates using window functions (MAX(MAX(event_timestamp)) OVER()), eliminating unnecessary self-joins.
4. Data Hygiene: Explicitly filters out uncaptured or null ecommerce.transaction_id records to prevent skew from invalid conversion tags.

---

## 📊 Summary Output Schema

Running either model produces a clean, customer-level grain table:

| Column | Type | Description |
| :--- | :--- | :--- |
| customer_id | STRING | Stitched user identifier (user_id or user_pseudo_id). |
| total_revenue | NUMERIC | Total lifetime spend for the evaluation period. |
| total_transactions | INTEGER | Distinct transaction count (Frequency). |
| recency | INTEGER | Days elapsed since the customer's last order. |
| monetary | NUMERIC | Average transaction value (AOV). |
| rfm_recency | INTEGER | Recency score (1 to 5). |
| rfm_frequency | INTEGER | Frequency score (1 to 5). |
| rfm_monetary | INTEGER | Monetary score (1 to 5). |
| segment | STRING | Assigned RFM behavioral segment label. |

---

## 🛠 Prerequisites & Usage

1. Replace the table wildcard path enter.tablename_123456.events_* with your Google Analytics 4 BigQuery export table ID.
2. Adjust the date interval in _TABLE_SUFFIX (default: 24 months) to match your dataset's historical depth.
3. Execute directly in BigQuery or schedule as a persistent table/view for Looker Studio or Power BI downstream reporting.
