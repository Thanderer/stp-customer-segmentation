# Data Quality Report — UCI Online Retail Dataset

**Author:** Ayush | **Phase:** 2 — Data & EDA | **Date:** June 2026

## 1. Dataset Overview

| Property | Value |
|---|---|
| Source | UCI Machine Learning Repository — Online Retail Dataset |
| File | `data/raw/online_retail.csv` (DVC tracked) |
| Raw rows | 541,909 transactions |
| Columns | 8 |
| Time period | December 2010 — December 2011 |
| Business context | UK-based online retailer |

## 2. Column Descriptions

| Column | Type | Description |
|---|---|---|
| `InvoiceNo` | String | Transaction ID — starts with 'C' if cancelled |
| `StockCode` | String | Product identifier |
| `Description` | String | Product name |
| `Quantity` | Integer | Units purchased — negative if cancelled/returned |
| `InvoiceDate` | DateTime | Date and time of transaction |
| `UnitPrice` | Float | Price per unit in GBP |
| `CustomerID` | Float | Unique customer identifier (~25% null) |
| `Country` | String | Customer country |

## 3. Data Quality Issues Found

### 3.1 Cancelled Orders
- **Issue:** Transactions where `InvoiceNo` starts with `'C'` represent cancellations
- **Volume:** Present throughout the dataset
- **Action:** Removed — cancelled orders do not reflect real purchase behaviour
- **Impact:** These rows have negative `Quantity` values and skew RFM calculations

### 3.2 Missing CustomerID
- **Issue:** Approximately 25% of rows have no `CustomerID`
- **Volume:** ~135,080 rows
- **Action:** Removed — customer-level RFM aggregation is impossible without an identifier
- **Justification:** No imputation strategy is viable since CustomerID is a key, not a measurable feature

### 3.3 Duplicate Rows
- **Issue:** Exact duplicate rows present in the dataset
- **Action:** Removed via `drop_duplicates()` — exact row matches are data entry errors

### 3.4 Invalid Quantity Values
- **Issue:** Rows with `Quantity <= 0` that are not cancellations
- **Action:** Removed — zero or negative quantity has no meaningful business interpretation for RFM

### 3.5 Invalid UnitPrice Values
- **Issue:** Rows with `UnitPrice <= 0`
- **Action:** Removed — free or negative-price items distort Monetary calculations

## 4. Cleaning Results

| Step | Rows Before | Rows After | Removed |
|---|---|---|---|
| Raw dataset | 541,909 | 541,909 | 0 |
| Remove cancellations | 541,909 | ~530,000 | ~11,909 |
| Remove null CustomerID | ~530,000 | ~440,000 | ~90,000 |
| Remove duplicates | ~440,000 | ~430,000 | ~10,000 |
| Remove invalid Qty/Price | ~430,000 | 392,692 | ~37,308 |
| **Final** | **541,909** | **392,692** | **149,217** |

## 5. RFM Feature Quality

| Feature | Min | Max | Mean | Std | Nulls |
|---|---|---|---|---|---|
| Recency (scaled) | -0.82 | 1.19 | 0.07 | 0.59 | 0 |
| Frequency (scaled) | -0.37 | 3.87 | 0.22 | 0.62 | 0 |
| Monetary (scaled) | -2.93 | 3.58 | 0.05 | 0.75 | 0 |

- **No missing values** in final feature store
- **4,338 unique customers** in final dataset
- All features scaled around 0 — confirms RobustScaler applied correctly

## 6. Known Limitations

| Limitation | Detail |
|---|---|
| Lost customers | ~25% of transactions have no CustomerID and are excluded — these may represent guest checkouts |
| Single year of data | Dataset covers Dec 2010 — Dec 2011 only — seasonal patterns may affect RFM scores |
| UK-centric | Majority of customers are from the UK — international customers are a small minority |
| No demographic data | RFM only captures transactional behaviour — no age, gender, or location used |

## 7. Compliance Notes

| Area | Status |
|---|---|
| PII | CustomerID treated as pseudonymous — no names, emails, or addresses in dataset |
| GDPR | No PII committed to repository at any point (OR01) |
| Data licence | UCI dataset is publicly available for academic use |
| Fairness | No demographic or protected characteristics used in feature engineering |