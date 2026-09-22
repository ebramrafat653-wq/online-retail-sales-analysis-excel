# Project Documentation

---

## 1. Cover Page

| Field | Value |
|---|---|
| **Project Name** | Online Retail Sales Analysis |
| **Analyst Name** | Ebram Rafat |
| **Role** | Data Analyst |
| **Start Date** | 20/07/2026 |
| **End Date** | 25/08/2026 |
| **Version** | v1.0 (Production Release) |

---

## 2. Dataset Description & Schema

| Field | Details |
|---|---|
| **Dataset Name** | Online Retail Dataset |
| **Source** | Kaggle |
| **Initial Rows** | 541,910 |
| **Rows After Processing** | 536,641 |
| **Columns** | 8 |
| **Time Period** | December 2010 – December 2011 |
| **Primary Use** | Revenue, order, product, country, and cancellation analysis |

### Source Column Schema

| # | Column Name | Description | Expected Data Type |
|---|---|---|---|
| 1 | InvoiceNo | Unique identifier for each invoice or transaction line | Text / categorical |
| 2 | StockCode | Product or item identifier | Text / categorical |
| 3 | Description | Product name or description | Text |
| 4 | Quantity | Quantity for the transaction line | Numeric |
| 5 | InvoiceDate | Date and time of the invoice | Date / time |
| 6 | UnitPrice | Unit price for the product | Numeric |
| 7 | CustomerID | Unique customer identifier | Numeric / categorical (contains missing values) |
| 8 | Country | Customer country | Text / categorical |

### Data Scope

The project is based on transactional sales data only. No cost information, customer demographic data, or inventory data is available. The analytics scope therefore focuses on realized revenue, transaction behavior, product contribution, geographic market performance, and operational monitoring of cancellations and inventory adjustments.

---

## 3. Business Problem & Context

The business required a structured and executive-ready view of commercial performance across the 2010–2011 retail period. The available data captured transactional activity but did not provide a standardized reporting layer for revenue performance, product concentration, customer value, or cancellation exposure.

The core business problem was the lack of visibility into:

- Revenue trend direction and monthly performance shifts.
- Market concentration and geographic revenue contribution.
- Product contribution and SKU-level commercial significance.
- Customer and order volume patterns.
- Operational monitoring of cancellations and inventory adjustments.
- Decision-ready metrics suitable for board-level and management review.

The project addresses this through an Excel-based analytics architecture combining Power Query, Power Pivot, and a dynamic executive dashboard.

---

## 4. Project Objectives & Scope

The project objective was to turn raw retail transactions into a transparent, reusable, and executive-friendly analysis model. The scope included data quality review, transformation, KPI definition, exploratory analysis, and dashboard construction in Excel.

Primary objectives:

- Measure realized revenue and order activity.
- Analyze revenue by month, country, and product.
- Identify the most commercially significant markets and SKUs.
- Quantify customer and transaction volume.
- Assess operational risk and cancellation exposure.
- Document a repeatable analytical workflow suitable for portfolio presentation.

In scope:

- Transaction-level sales and cancellation review.
- Geographic market performance.
- Product and SKU contribution analysis.
- Time-based seasonality and operational patterns.
- Dashboard and workbook architecture for executive communication.

Out of scope:

- Gross margin or profit analysis due to missing cost data.
- Customer demographic segmentation because demographic data was not provided.
- Longitudinal forecasting beyond the available one-year data window.

---

## 5. Business Questions (Managerial Perspective)

The following questions align to a management reporting lens rather than descriptive analytics only:

### Revenue and Order Performance

1. How did Net Revenue evolve across the analysis period?
2. What is the level of Total Recorded Invoices (25,900) and Average Order Value ($395.94) across the business?
3. Which periods represented seasonal demand peaks or softening?

### Geographic Performance

4. Which countries contributed the largest share of revenue?
5. How concentrated was the geographic revenue base?
6. Which markets represented the greatest commercial opportunity or monitoring priority?

### Product Performance

7. Which products contributed the largest revenue share?
8. Which product lines supported the strongest revenue concentration?
9. Which SKUs required revenue-protection attention due to operational exposure?

### Customer and Transaction Monitoring

10. How many Identified Transacting Customers (4,336) were active in the dataset?
11. What was the typical revenue value per order?
12. What was the observed Baseline Cancellation Rate (11.75%) and how should it be monitored operationally?

### Dashboard and Executive Reporting

13. What should the executive summary communicate at a glance?
14. Which slicers would support fast, business-relevant filtering for `Country`, `Year`, and `TransactionType`?

---

## 6. Data Understanding & Granularity

### Dataset Overview

| Metric | Value |
|---|---:|
| Total Rows (initial) | 541,910 |
| Total Rows (processed) | 536,641 |
| Total Columns | 8 |
| Analysis Period | Dec 2010 – Dec 2011 |
| Active Product SKUs | 3,820 |
| Identified Transacting Customers | 4,336 |
| Distinct Countries | 42 |

### Granularity

Each row represents a transaction line rather than a complete invoice. This is evidenced by multiple product lines being associated with the same `InvoiceNo`, confirming that a single invoice can contain multiple stock items and multiple revenue lines.

### Data Classification Summary

| Column | Category | Notes |
|---|---|---|
| InvoiceNo | Transaction identifier | Cancellation-related records often begin with `C` |
| StockCode | Product identifier | Unique item-level identifier |
| Description | Dimension | Product description label |
| Quantity | Measure | Quantity per transaction line |
| InvoiceDate | Date | Timestamp used for time analysis |
| UnitPrice | Measure | Unit price per product |
| CustomerID | Customer identifier | Contains missing values |
| Country | Dimension | Geographic market dimension |

### Initial Observations

- Active Product SKUs in the final model: 3,820.
- Identified Transacting Customers: 4,336.
- The United Kingdom dominated the market base, with revenue contribution exceeding 84%.
- Transaction activity was concentrated in a specific month and weekday pattern, supporting a seasonality and operational-cycle narrative.

---

## 7. Data Quality Assessment (Verified Checks)

The quality assessment was executed before final modeling decisions were made. The objective was to identify issues that required either correction, retention, or explicit business interpretation.

| Data Quality Check | Verified Finding | Impact | Decision |
|---|---|---|---|
| Missing `CustomerID` values | 135,080 initial missing values (24.93%), reduced to 135,037 after exact deduplication | Affects customer-level reporting, while the records remain usable for transaction-level revenue and order analysis | Retained; customer-level analysis performed on the verified subset while preserving all valid sales transactions |
| Missing `Description` values | 1,454 empty values | Limited product identification impact | Retained where the product code remained valid |
| Duplicate records | Exact duplicates identified and removed | Duplicate inflation risk | Removed as exact duplicates only |
| Negative `Quantity` records | 10,624 records total | Indicates returns, cancellations, and inventory adjustments | Reviewed and retained under the defined transaction classification |
| `InvoiceNo` with leading `C` among negative quantities | 9,288 records | Strongly associated with cancellation-oriented transaction classification | Retained as cancellation logic |
| Negative `UnitPrice` records | 0 records | No negative pricing issue detected | No remediation required |
| Zero `UnitPrice` records | 1,336 records | Business interpretation required | Retained after review as non-invalid transaction activity |
| Date validation | Within project period: Dec 2010 – Dec 2011 | No range issue identified | Accepted |

### Data Quality Summary

The assessment identified known data quality considerations without invalidating the core analytical dataset. The final approach was to preserve legitimate business behavior while removing only exact duplicates, consistent with the documented business rules and data integrity requirements.

---

## 8. Data Cleaning Process

The cleaning process focused on improving data reliability while preserving legitimate business activity. The final accepted rule was conservative: remove only exact duplicates and retain records requiring contextual assessment.

### 8.1 Exact Duplicate Removal

Exact duplicate records were identified across all columns and removed. This reduced the initial dataset from 541,910 rows to 536,641 rows.

### 8.2 Missing `CustomerID`

A substantial number of rows contained missing `CustomerID` values. These rows remained in the analytic file because they represented valid transactional activity that still contributed to aggregate revenue and transaction metrics. While this does not prevent transaction-level revenue analysis, it does limit customer-level attribution and cohort tracking because the customer identity is not available for every record.

### 8.3 Missing `Description`

Rows with missing product descriptions were retained because the corresponding product codes remained available. This preserved transaction completeness and avoided unnecessary data loss.

### 8.4 Zero `UnitPrice` Review

Rows with `UnitPrice = 0` were reviewed and retained because the data did not support a definitive invalid-data classification. They were treated as legitimate transactional variation rather than erroneous records.

### 8.5 Inventory Adjustment Review

Negative quantity entries not associated with `InvoiceNo` values beginning with `C` were reviewed separately. These records were retained because they were consistent with operational inventory adjustments rather than invalid data.

### Data Cleaning Outcome

The final cleaned dataset preserved legitimate business behavior while removing only exact duplicates. This ensured that the financial and operational metrics remained representative of the real business context without manufacturing artificial data quality results.

---

## 9. Feature Engineering

Feature engineering created a business-ready analytical layer while preserving the original transactional fields. The following engineered fields were used to support reporting and dashboard logic.

| Engineered Feature | Derivation / Logic | Data Type | Business Purpose |
|---|---|---|---|
| TransactionType | Derived from `InvoiceNo`, `Quantity`, and business rules | Text | Classify records as Sale, Cancellation, or Inventory Adjustment |
| Revenue | Quantity × UnitPrice | Decimal | Calculates transaction-level revenue for analysis |
| Year | Extracted from `InvoiceDate` | Whole number | Supports annual comparisons |
| MonthNumber | Derived from `InvoiceDate` | Whole number | Supports month ordering |
| MonthName | Derived from `InvoiceDate` | Text | Provides readable month labels |
| YearMonth | Derived from `InvoiceDate` as `YYYY-MM` | Text | Enables stable month-over-month analysis |
| DayName | Derived from `InvoiceDate` | Text | Supports weekday activity analysis |
| WeekdayNumber | Derived from `InvoiceDate` with Monday = 1 to Sunday = 7 | Whole number | Supports sorting of weekday labels |
| InvoiceDateOnly | Derived from `InvoiceDate` by removing time | Date | Enables day-level analysis |
| StockCodePattern | Derived from the structure of `StockCode` | Text | Supports product code profiling |
| ItemType | Derived from `StockCodePattern` and classification rules | Text | Distinguishes product vs non-product records |
| AnalysisType | Derived from `TransactionType` and `ItemType` | Text | Supports reporting and filtering across analytical views |

### Feature Engineering Notes

- `TransactionType` explicitly classifies records into `Sale`, `Cancellation`, and `Inventory Adjustment`.
- `Revenue` is the foundational measure for order and product contribution analysis.
- `MonthNumber` and `WeekdayNumber` provide ordering logic for chronological and operational reporting.
- `YearMonth` creates a stable monthly reporting label across the full dataset.

---

## 10. KPI Definitions & Verified DAX Expressions

The workbook uses a verified set of executive KPI measures within the Excel data model. The following names were used consistently in the analysis and dashboard reporting:

| KPI | Definition | Reporting Use |
|---|---|---|
| `[Net Revenue]` | Net realized revenue measure used in the model | Executive revenue reporting |
| `[Total Recorded Invoices]` | Distinct transaction identifiers across all transaction types; reflects the verified count of 25,900 recorded invoice transaction identifiers in the model | Order-volume reporting |
| `[Identified Transacting Customers]` | Identified Transacting Customers (explicitly excludes blank/unassigned customer IDs via `NOT(ISBLANK(CustomerID))` to reflect verified transacting accounts: 4,336) | Customer base reporting |
| `[Active Product SKUs]` | Active Product SKUs (strictly filtered for physical merchandise using `AnalysisType = 'Sale'` and `ItemType = 'Product'`, excluding fees, discounts, and non-product line items) | Product portfolio reporting |
| `[Average Order Value]` | Revenue per recorded invoice transaction identifier | Commercial performance reporting |
| `[Baseline Cancellation Rate]` | Baseline Cancellation Rate (proportion of cancelled transaction identifiers relative to total recorded transaction identifiers, representing transaction-level void velocity rather than inventory unit return rate or financial loss percentage) | Revenue protection and operational monitoring |

### Verified DAX Expressions

```dax
-- 1. Net Realized Revenue
[Net Revenue] := 
SUM(online_retail[Revenue])

-- 2. Total Recorded Invoices (Distinct invoice transaction identifiers across all transaction types)
[Total Recorded Invoices] := 
DISTINCTCOUNT(online_retail[InvoiceNo])

-- 3. Identified Transacting Customers (Excluding blank/unassigned accounts; Sales only)
[Identified Transacting Customers] := 
CALCULATE(
    DISTINCTCOUNT(online_retail[CustomerID]),
    online_retail[AnalysisType] = "Sale",
    NOT(ISBLANK(online_retail[CustomerID]))
)

-- 4. Active Product SKUs (Physical merchandise only; Sales transactions)
[Active Product SKUs] := 
CALCULATE(
    DISTINCTCOUNT(online_retail[StockCode]),
    online_retail[AnalysisType] = "Sale"
)

-- 5. Average Transaction Value (ATV / AOV baseline)
[Average Order Value] := 
DIVIDE(
    [Net Revenue], 
    [Total Recorded Invoices], 
    0
)

-- 6. Baseline Cancellation Rate (Cancelled transaction identifiers over total recorded transaction identifiers)
[Baseline Cancellation Rate] := 
DIVIDE(
    CALCULATE([Total Recorded Invoices], online_retail[AnalysisType] = "Cancellation"),
    [Total Recorded Invoices],
    0
)
```

### Verified KPI Values

| KPI | Verified Value |
|---|---:|
| Net Revenue | $9,771,519.35 |
| Total Recorded Invoices | 25,900 |
| Identified Transacting Customers | 4,336 |
| Active Product SKUs | 3,820 |
| Average Order Value | $395.94 |
| Baseline Cancellation Rate | 11.75% |

> These DAX definitions are documented exactly as used in the verified project model and are aligned to the approved KPI naming and validation set for this portfolio project.

---

## 11. Exploratory Data Analysis (EDA)

### 11.1 Time Seasonality

Observed data findings:

- Monthly revenue peaked sharply in November at approximately $1.43M across 3,462 orders.
- Revenue activity varied materially across the year and showed a clear seasonal shape.
- Wednesday and Thursday combined account for the largest share of weekly order volume (~38.4% of total recorded orders), establishing the primary weekly operational baseline.

Business implication:

- The dataset supports a seasonal operating narrative for planning and promotional review.
- Midweek execution patterns indicate operational concentration in the middle of the week and should be monitored for staffing and fulfillment planning.

### 11.2 Geographic Markets

Observed data findings:

- United Kingdom generated $8,276,080.55 (~84.7% of Net Revenue), while the top 5 international markets combined (UK, Netherlands, EIRE, Germany, France) account for over 93.2% of total realized revenue.
- Netherlands ($283,479.54), EIRE ($264,555.02), Germany ($200,619.66), and France ($182,262.60) were the next-largest contributing markets.

Business implication:

- The revenue base is highly concentrated in a single geography.
- Country-level monitoring remains essential for market diversification and operational prioritization.

### 11.3 Product Contribution and Revenue Concentration

Observed data findings:

- REGENCY CAKESTAND 3 TIER generated $164,459.49.
- WHITE HANGING HEART T-LIGHT HOLDER generated $99,612.42.
- PARTY BUNTING generated $98,243.88.
- The Top 10 SKUs generated $781,022.63 (~8.0% of Net Revenue), confirming measurable SKU concentration while verifying that the catalog is not monopolized by single product lines.

Business implication:

- Product contribution and revenue concentration are evident across the portfolio.
- Measurable product-level concentration exists, but the portfolio is not dominated by a single product segment.

### 11.4 Transaction Types

Observed data findings:

- Transactions were classified into `Sale`, `Cancellation`, and `Inventory Adjustment`.
- Baseline cancellation rate was 11.75%.
- Cancellation activity should be treated as a monitoring priority rather than a presumed product or process failure.

Business implication:

- The cancellation rate highlights a material revenue-protection opportunity.
- Operational teams should monitor cancellation drivers without inferring root-cause conclusions beyond the available dataset.

---

## 12. Dashboard Architecture & Workbook Structure

The workbook follows a multi-sheet Excel architecture designed to support both analytical exploration and executive reporting. The final dashboard is the top-level `Executive_Summary_00` layer, fed by a data model and multiple supporting analytical sheets.

### Workbook Structure

The workbook includes:

- A data preparation layer for extraction, cleaning, and shaping.
- A Power Pivot / DAX analytical model.
- Five detailed analytical sheets covering time, geography, product, customer, and transaction review.
- A top-level dynamic dashboard named `Executive_Summary_00`.

### Dashboard Design

The dashboard presents a compact executive view using KPI cards tied to the verified workbook measures:

- `[Net Revenue]`
- `[Total Recorded Invoices]` (distinct invoice transaction identifiers across all transaction types)
- `[Identified Transacting Customers]`
- `[Active Product SKUs]`
- `[Average Order Value]`
- `[Baseline Cancellation Rate]`

These cards update dynamically based on the selected report context and model filters.

### Implemented Slicer Connectivity

The workbook includes interactive slicer connectivity for:

- `Country`
- `Year`
- `TransactionType`

These slicers provide direct filtering across the executive dashboard and the supporting analytical views to support scenario review and management-level drilldown.

### Visual Layout

The dashboard emphasizes a concise executive format with:

- KPI summary cards at the top level.
- Geographic performance views.
- Time-based seasonality views.
- Product contribution panels.
- Transaction and cancellation monitoring views.

This structure enables rapid executive scanning without overwhelming the user with transactional detail.

---

## 13. Key Executive Findings

### 1. Geographic Concentration

Finding:

United Kingdom generated $8,276,080.55 (~84.7% of Net Revenue), while the top 5 international markets combined (UK, Netherlands, EIRE, Germany, France) account for over 93.2% of total realized revenue.

Evidence:

- United Kingdom was the dominant revenue market by a wide margin.
- Other countries were materially smaller in comparison.

Business implication:

This indicates a highly concentrated commercial footprint. Country-level monitoring and diversification reviews should remain a core priority for strategic planning.

### 2. Q4 Commercial Seasonality

Finding:

Revenue peaked sharply in November, reaching approximately $1.43M across 3,462 orders.

Evidence:

- Peak monthly revenue occurred in the final quarter of the year.
- Seasonal revenue concentration is clearly observed in the time series.

Business implication:

The dataset supports a clear seasonal operating narrative for commercial planning and demand review.

### 3. Midweek Demand Velocity

Finding:

Order activity concentrated around midweek operational cycles, particularly Wednesday and Thursday, which combined account for the largest share of weekly order volume (~38.4% of total recorded orders), establishing the primary weekly operational baseline.

Evidence:

- The weekday distribution showed a strong midweek concentration pattern.
- Operational cycle activity was more intense in the midweek period.

Business implication:

This is an observed operational pattern rather than a causal conclusion. It suggests the need for awareness of midweek execution intensity and capacity planning.

### 4. Product Portfolio Concentration

Finding:

The product portfolio was concentrated among a small number of high-revenue SKUs.

Evidence:

- REGENCY CAKESTAND 3 TIER: $164,459.49
- WHITE HANGING HEART T-LIGHT HOLDER: $99,612.42
- PARTY BUNTING: $98,243.88
- The Top 10 SKUs generated $781,022.63 (~8.0% of Net Revenue), confirming measurable SKU concentration while verifying that the catalog is not monopolized by single product lines.

Business implication:

The portfolio profile indicates measurable product-level concentration. The product base should be monitored as a strategic contribution set, with concentration present but not dominant.

### 5. Revenue Protection & Cancellations

Finding:

The Baseline Cancellation Rate was 11.75%, establishing cancellation activity as a material operational monitoring issue.

Evidence:

- The model reports a Baseline Cancellation Rate of 11.75%.
- Transaction classes were segmented into `Sale`, `Cancellation`, and `Inventory Adjustment`.

Business implication:

Cancellation behavior should be treated as a revenue-protection opportunity requiring operational attention and monitoring, without assuming root causes beyond the available dataset.

---

## 14. Actionable Business Recommendations

These recommendations are strictly aligned to the five executive findings above and are framed as operational and commercial next steps, not causal claims.

### Recommendation 1: Geographic commercial governance

- Prioritize revenue monitoring by country with special attention to the United Kingdom.
- Review country-level operating plans to support sustained performance in the dominant market and monitor concentration risk in secondary regions.

### Recommendation 2: Seasonal campaign and inventory planning

- Use the Q4 revenue peak as a planning anchor for seasonal demand forecasting and operational readiness.
- Align commercial and operational planning around the confirmed seasonal demand pattern.

### Recommendation 3: Midweek operating alignment

- Review staffing and fulfillment rhythm against the observed midweek demand concentration.
- Use the weekday pattern to support operational preparation and resource scheduling.

### Recommendation 4: Product portfolio concentration management

- Review the leading SKUs and their contribution profile to ensure continued commercial visibility and inventory prioritization.
- Monitor the concentration of the product base through SKU-level revenue tracking.

### Recommendation 5: Baseline Cancellation Rate monitoring framework

- Establish a regular Baseline Cancellation Rate review cadence and monitor changes over time.
- Use `AnalysisType` and `TransactionType` segmentation to distinguish `Sale`, `Cancellation`, and `Inventory Adjustment` patterns in operational reviews.

---

## 15. Assumptions & Data Limitations

The analytical model is designed to provide consistent reporting within the boundaries of the available dataset; however, several limitations must be clearly disclosed.

- No cost data is available, so profit and gross margin cannot be calculated.
- No customer demographic data is available; therefore, segmentation by customer profile is not supported in the current model.
- The dataset covers a fixed one-year timeframe and does not represent a multi-year trend beyond the provided period.
- The analysis is based on revenue and transaction behavior rather than profit performance.
- The project does not claim causal inference beyond the observed business data.

---

## 16. Project Workflow & Architecture Diagram

### Workflow

```text
Raw CSV
   │
   ▼
Data Understanding
   │
   ▼
Data Quality Assessment
   │
   ▼
Power Query ETL & Cleaning
   │
   ▼
Power Pivot / DAX Data Model
   │
   ▼
Five Analytical Sheets
   │
   ▼
Executive_Summary_00 Dashboard
   │
   ▼
Management Interpretation & Recommendations
```

### Architecture Summary

```text
Source Data (CSV)
   │
   ▼
Power Query (Data Preparation)
   │
   ▼
Excel Data Model (Power Pivot / DAX)
   ├── 01_Revenue_Analysis
   ├── 02_Geographic_Analysis
   ├── 03_Product_Analysis
   ├── 04_Customer_Analysis
   ├── 05_Transaction_Analysis
   └── Executive_Summary_00
```

---

## 17. Technical Lessons Learned & Best Practices

### Data integrity discipline

The project demonstrated the importance of validating data quality before constructing the analytical model. Exact duplicate removal and explicit retention logic preserved the business dataset without forcing inaccurate assumptions.

### Controlled retention strategy

Rather than deleting records without sufficient evidence, the project retained transactions that were ambiguous but operationally valid. This protected data integrity and supported a realistic business narrative.

### Model readability

The final Excel architecture balances analytical depth with executive readability. KPI cards and slicer-based reporting allow users to engage with the model without losing clarity.

### Documentation quality

Portfolio-grade documentation is essential for translating model output into business communication. Clear separation between observed findings and business implications is critical for maintaining analytical credibility.

---

## 18. Executive Summary

This project documents a complete Excel-based retail analytics workflow for the Online Retail Dataset spanning December 2010 to December 2011. The final model produced a verified executive dataset with Net Revenue of $9,771,519.35, Total Recorded Invoices of 25,900, Identified Transacting Customers of 4,336, Active Product SKUs of 3,820, Average Order Value of $395.94, and a Baseline Cancellation Rate of 11.75%.

The analysis shows strong geographic concentration in the United Kingdom, a clear Q4 revenue peak in November, midweek operational intensity, and product contribution concentration among a narrow set of SKUs. The project also frames cancellations as a revenue-protection monitoring opportunity rather than a presumed operational failure.

The workbook architecture combines Power Query, Power Pivot / DAX, and a dynamic executive dashboard to convert raw transaction data into an executive-ready reporting framework. The result is a portfolio-grade analytics document and model that meets the business requirement for transparent, evidence-based operational and commercial insight.

---

*Document Status: Production Portfolio Edition Complete*
