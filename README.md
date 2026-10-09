# OVERVIEW

BigQuery SQL models for performing Recency, Frequency, and Monetary (RFM) Customer Segmentation using Google Analytics 4 (GA4) export data.

## REPOSITORY

### 1. 1.0_RFM-Discrete-Rules-Option-A.sql (Fixed Lookup Model)
* Approach: Uses discrete integer scoring (1 to 5) and a standard 11-segment lookup mapping.
* Best Used For: Matching traditional categorical RFM frameworks, standardized BI dashboards, and step-by-step reporting walkthroughs.
* Key Logic: Concatenates rfm_recency, rfm_frequency, and rfm_monetary into a 3-character string (e.g., '555', '511') and maps it against an explicit lookup CASE statement.

### 2. 1.1_RFM-Percentile-Dynamic-Option-B.sql (Production Model)
* Approach: Continuous mathematical percentiles (PERCENT_RANK()) combined with composite threshold scoring.
* Best Used For: Scaled production pipelines and stores with non-standard order distributions (e.g., high AOV low-frequency stores, or subscription models).
* Key Logic: Replaces static lookup strings with fluid threshold rules (rfm_recency >= 4 AND rfm_frequency >= 4), ensuring 100% of customers fall into active behavioral segments without edge-case NULL values.

### 3. 1.2_Days-Between-First-And-Last-Purchase.sql (Lifecycle Auxiliary)
* Approach: Summary query computing customer lifespan metrics.
* Key Metrics: Orders count, total spend, AOV, first purchase date, most recent purchase date, and total active days elapsed between orders.

## SQL OPTIMISATION & PERFORMANCE 

All models in this repository incorporate key performance and data-quality optimizations:

1. User Identity Stitching: Utilizes COALESCE(user_id, user_pseudo_id) to unify guest browsing cookies with authenticated account logins, preventing duplicate single-purchase profiles.
2. Partition Pruning: Filters _TABLE_SUFFIX >= FORMAT_DATE('%Y%m%d', DATE_SUB(CURRENT_DATE(), INTERVAL 24 MONTH)) to limit BigQuery byte-scans to active evaluation windows.
3. Single-Pass Aggregations: Derives dataset-wide reference dates using window functions (MAX(MAX(event_timestamp)) OVER()), eliminating unnecessary self-joins.
4. Data Hygiene: Explicitly filters out uncaptured or null ecommerce.transaction_id records to prevent skew from invalid conversion tags.

## SUMMARY OUTPUT SCHEME

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


