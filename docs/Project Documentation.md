# Project Documentation

---

## 1. Cover Page

| Field | Value |
|---|---|
| **Project Name** | Online Retail Sales Analysis |
| **Analyst Name** | *[Ebram Rafat]* |
| **Role** | Data Analyst |
| **Start Date** | *[20/7/2026]* |
| **End Date** | *[Target Date]* |
| **Version** | v0.1 (Draft) |

---

## 2. Dataset Description

| Field | Details |
|---|---|
| **Dataset Name** | Online Retail Dataset |
| **Source** | Kaggle |
| **Number of Rows** | 541,910 |
| **Number of Columns** | 8 |
| **Time Period** | December 2010 – December 2011 |

### Column Description

| # | Column Name | Description | Expected Data Type |
|---|---|---|---|
| 1 | InvoiceNo | Unique identifier for each invoice/transaction | Text / Categorical (Note: cancelled orders often start with "C") |
| 2 | StockCode | Unique product/item code | Text / Categorical |
| 3 | Description | Name/description of the product | Text |
| 4 | Quantity | Number of units purchased per transaction line | Numeric (Integer) |
| 5 | InvoiceDate | Date and time the invoice was generated | Date/Time |
| 6 | UnitPrice | Price per unit of the product | Numeric (Decimal) |
| 7 | CustomerID | Unique identifier for each customer | Numeric / Categorical (contains missing values) |
| 8 | Country | Country where the customer is located | Text / Categorical |

> Note on Scope: This project is based on transactional sales data only. No cost data, customer demographic data, or inventory data are available; therefore, the analysis will focus on revenue, order behavior, product performance, and customer activity.

---

## 3. Business Problem

The company has accumulated transactional sales data from Dec 2010 to Dec 2011, but it lacks clear visibility into business performance and growth drivers. Management needs a structured analysis to better understand sales trends, customer value, product contribution, geographic performance, and operational losses caused by cancellations and returns.

Key business challenges include:

- **Lack of Sales Visibility:** Limited understanding of revenue trends and monthly growth patterns.
- **Customer Insights Gap:** Difficulty in identifying high-value customers and purchase behavior.
- **Product & Market Performance:** Need to identify top-performing and low-performing products and countries.
- **Operational Leakage:** Unmonitored cancellations and returns can affect net revenue.
- **Decision-Making Support:** Need for a centralized dashboard to support strategic actions.

---

## 4. Project Objectives

The primary objectives of this project are to:

- Evaluate sales performance using revenue, order volume, and key business metrics.
- Analyze monthly revenue trends and growth patterns over time.
- Identify the best- and worst-performing products by revenue and sales volume.
- Understand customer purchasing patterns and highlight high-value customers.
- Assess geo-market performance across countries.
- Measure cancellation and return patterns to highlight potential revenue leakage.
- Detect seasonality and demand fluctuations.
- Design an interactive Power BI dashboard for ongoing business monitoring.
- Deliver actionable business insights and recommendations.

---

## 5. Business Questions

> These questions are framed from a senior business analyst perspective and are intended to support managerial decision-making rather than only describe the dataset.

>Return/Cancellation is defined consistently across all questions; see KPI Definitions (Section 9) for the exact rule.

### Revenue Performance

1. How did total monthly revenue evolve throughout the analysis period (Dec 2010 – Dec 2011)?
2. How does monthly revenue change compared to the previous month (Month-over-Month Revenue Growth %)?
3. What are the peak and low sales seasons based on monthly revenue?

### Geographic Analysis

4. How is total revenue distributed across countries, and which countries generate the highest and lowest revenue?
5. Which countries have the highest cancellation/return rates?

### Product Performance

6. Which products generate the highest revenue?
7. Which products have the highest sales volume (Quantity Sold)?
8. Which products have the highest return rates?
9. Which products contribute the least to total revenue?

### Customer Analysis

10. Who are the top customers based on total revenue?
11. Which customers place the highest number of orders?
12. Which customers have the highest cancellation/return rates?

### Order Analysis

13. What is the average order value (AOV)?
14. What percentage of all orders are cancelled or returned?
15. How do sales patterns vary across months and days of the week?

---

## 6. Data Understanding

### Dataset Overview

| Metric | Value |
|---|---|
| Total Rows | 541,910 |
| Total Columns | 8 |
| Analysis Period | Dec 2010 – Dec 2011 |
| Unique Products (StockCode) | 3,962 |
| Unique Customers (CustomerID) | 4,377 |
| Distinct Countries | 42 |

### Data Granularity

Data granularity was determined by analyzing the relationship between InvoiceNo and StockCode. The same InvoiceNo can appear across multiple rows with different StockCode values, indicating that each row represents a single product line within an invoice rather than a complete invoice. A single invoice may therefore contain multiple products purchased by the same customer.

### Column Classification

| Column | Category | Notes |
|---|---|---|
| InvoiceNo | Transaction Identifier | Cancelled invoices are identified by  an InvoiceNo starting  with "C" |
| StockCode | Product Identifier | Product-level identifier |
| Description | Dimension | Product name/description |
| Quantity | Measure | Units per transaction line |
| InvoiceDate | Date | Transaction timestamp |
| UnitPrice | Measure | Price per unit |
| CustomerID | Customer Identifier | Contains missing values that will be evaluated during the Data Quality Assessment phase |
| Country | Dimension | Customer location |

### Initial Observations

- The dataset contains 3,962 unique products sold across the analysis period.
- The dataset contains 4,377 distinct CustomerID values; missing CustomerID records will be assessed during Data Quality Assessment.
- Sales transactions span 42 countries, indicating an international customer base.
- Each row represents a single product line within an invoice, not a complete invoice.

---

## 7. Data Quality Assessment

The objective of this phase is to identify data quality issues before any cleaning actions are performed. This assessment documents the verified findings observed in the dataset and highlights the next actions required to evaluate potential data quality concerns.

| Data Quality Check | Finding | Impact Assessment | Recommended Action |
|---|---|---|---|
| Missing Values | CustomerID has 135,080 missing values (24.93%). Description contains 1,454 empty strings (0.27%). | Missing values are concentrated in CustomerID, while Description contains a small number of empty strings. Their impact will be evaluated during the data cleaning phase. | Investigate the source of these missing values and define the appropriate handling approach for each field. |
| Duplicate Records | Potential duplicate groups were detected using Group By (all columns) + Count Rows, resulting in 4,879 duplicate groups. | Potential duplicate groups were identified. Their business validity must be confirmed before any deduplication is performed. | Validate the duplicate groups and document the business rule for handling them before any action is taken. |
| Quantity Validation | Negative Quantity records total 10,624. Of these, 9,288 (87.43%) have InvoiceNo values starting with "C", while 1,336 (12.57%) do not start with "C". | The distribution suggests a need for further review of quantity anomalies and their relationship to cancellations or returns. | Investigate the negative quantity records and confirm the appropriate business interpretation before defining the final rule. |
| UnitPrice Validation | Negative UnitPrice records: 0. Zero UnitPrice records: 1,336. | No negative unit prices were detected, but zero-price records require review. | Investigate the zero-price records and determine whether they should be included in the final business rule review. |
| InvoiceDate Validation | The invoice date range is valid and consistent with the project scope: December 2010 through December 2011. | The date field appears consistent with the expected project period. | Continue using the date range as a reference point while investigating the other data quality issues. |

Overall, the assessment identified several areas requiring further review, particularly missing values, duplicate groups, quantity anomalies, and zero-price records. The invoice date field appears consistent with the defined project scope, and no negative UnitPrice values were detected.

---

## 8. Data Cleaning Process

The objective of this phase was to improve data reliability while preserving as much valid business information as possible. Cleaning decisions were based on data validation rather than assumptions to avoid removing legitimate transactions.

### 8.1 Data Type Validation

All columns were reviewed, and their data types were verified before performing any transformation to ensure subsequent operations were applied correctly.

### 8.2 Exact Duplicate Removal

Potential duplicate records were initially identified during the Data Quality Assessment using Group By across all columns.

A sample of duplicate groups was manually reviewed to confirm that the duplicated records were identical across every column. After validation, Remove Duplicates was applied using all columns.

Cleaning Summary

- Initial rows: 541,910
- Duplicate rows removed: 5,269
- Final rows after deduplication: 536,641

### 8.3 Missing CustomerID

After duplicate removal, 135,037 records (approximately 25.16%) still contained missing CustomerID values.

These records were retained because they contain valid transactional information (InvoiceNo, StockCode, Quantity, UnitPrice, and InvoiceDate). Missing CustomerID only affects customer-level analysis and does not impact sales analysis.

Decision: Retained.

### 8.4 Missing Description

A total of 1,454 records contained missing Description values.

These records were retained because the corresponding StockCode remained available, allowing products to be identified where necessary.

Decision: Retained.

### 8.5 Zero UnitPrice Validation

A total of 2,510 records had UnitPrice = 0.

Further investigation showed that these records were not limited to inventory adjustments. Some contained positive quantities, suggesting that they may represent legitimate business events such as promotional items, free samples, or internal transactions.

Since there was insufficient evidence to classify these records as invalid data, they were retained.

Decision: Retained.

### 8.6 Inventory Adjustment Investigation

Negative quantity records without an InvoiceNo beginning with "C" were investigated separately.

These records showed characteristics different from standard customer cancellations, including:

- UnitPrice = 0
- Descriptions such as Damage, Broken, Missing, and similar operational terms
- Transactions distributed throughout the analysis period

Rather than removing these records, they were retained because they appear to represent legitimate inventory adjustments rather than data quality issues.

Decision: Retained.

Cleaning Outcome

Only exact duplicate records were removed during the cleaning process. All other identified issues were investigated individually and retained because they represent valid business scenarios rather than invalid data.

---

## 9. Feature Engineering

Feature engineering was performed after Data Cleaning to create a concise set of analytical fields that simplify transactional reporting, support time-based analysis, and provide consistent business classifications for analysis and dashboard development. All engineered fields are derived from existing transactional columns, while the original transactional data remains preserved.

| Engineered Feature | Derivation / Logic | Data Type | Business Purpose |
|---|---:|---|---|
| TransactionType | Derived from `InvoiceNo`, `Quantity`, and transaction business rules. Values: "Sale", "Cancellation", "Inventory Adjustment". | Text (enumeration) | Classify transactions consistently for sales, cancellations/returns, and inventory adjustment analysis. |
| Revenue | Formula: "[Quantity] * [UnitPrice]" | Decimal Number | Calculate transaction-level revenue and support revenue and contribution analysis. |
| Year | Derived from `InvoiceDate` using the calendar year. | Whole Number | Support yearly filtering and time-based analysis. |
| MonthNumber | Derived from `InvoiceDate` as a value from 1 to 12. | Whole Number | Preserve chronological month ordering and serve as a sorting key for `MonthName`. |
| MonthName | Derived from `InvoiceDate` as the English month name. | Text | Provide readable month labels for reports, Pivot Tables, and dashboards. |
| YearMonth | Derived from `InvoiceDate` and formatted as `YYYY-MM` (e.g., "2011-03"). | Text | Provide a stable monthly analytical label for chronological trend analysis. |
| DayName | Derived from `InvoiceDate` as the English weekday name (Monday–Sunday). | Text | Support analysis of transaction and sales patterns by day of week. |
| WeekdayNumber | Derived from `InvoiceDate`, with Monday = 1 through Sunday = 7. | Whole Number | Serve as a sorting key for `DayName` to maintain chronological weekday order. |
| InvoiceDateOnly | Derived from `InvoiceDate` by removing the time component and retaining only the calendar date. | Date | Support daily-level analysis while avoiding time-of-day granularity when not required. |
| StockCodePattern | Derived from the structure of `StockCode` to classify stock codes by character pattern (for example: "Numeric", "Mixed", "Text"). | Text (enumeration) | Support product-code profiling and provide a structural basis for distinguishing standard product codes from special or non-product codes. |
| ItemType | Derived from `StockCodePattern` and business classification rules. Values: "Product", "Non-Product". | Text (enumeration) | Distinguish physical product items from service, fee, discount, and other non-product transaction items. |
| AnalysisType | Derived from `TransactionType` and `ItemType` to provide an analysis-oriented classification. Product transactions are classified by sale/cancellation while non-product records are grouped separately. | Text (enumeration) | Provide a consistent analytical layer for filtering transactions and separating product sales, product cancellations, and non-product records during analysis. |

These engineered features improve analytical flexibility by simplifying transactional analysis, supporting time-based analysis, and providing consistent business classifications.

`MonthNumber` and `WeekdayNumber` serve as supporting sort keys for `MonthName` and `DayName`, respectively. `YearMonth` provides the primary monthly label for chronological trend analysis, while `InvoiceDateOnly` supports daily-level analysis without unnecessary time granularity.

`StockCodePattern`, `ItemType`, and `AnalysisType` provide complementary classification layers: `StockCodePattern` describes the structural pattern of the stock code, `ItemType` identifies whether the record represents a product or non-product item, and `AnalysisType` provides the final classification used for analytical filtering and reporting.

All engineered fields are derived from existing transactional data, and the original source columns remain preserved.

---

## 10. KPI Definitions

*[To be added later]*

- Total Revenue
- Total Orders
- Average Order Value
- Return Rate
- Revenue Growth

---

## 10. Exploratory Data Analysis (EDA)

*[To be added later]*

- Summary Statistics
- Revenue Distribution
- Customer Analysis
- Product Analysis
- Country Analysis
- Seasonal Analysis

---

## 11. Dashboard Design

*[To be defined before building the Power BI dashboard]*

---

## 12. Key Findings

*[To be completed after analysis]*

---

## 13. Business Recommendations

*[This is a key section and will be written after the findings are finalized]*

---

## 14. Assumptions & Limitations

*[To be documented at the end of the project]*

- No cost data is available; therefore, profit cannot be calculated.
- No customer demographic data is available; analysis will focus on purchasing behavior.
- The dataset covers a specific time period and may not reflect future performance.

---

## 15. Project Workflow

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
Data Cleaning (Power Query)
   │
   ▼
EDA
   │
   ▼
Power BI Dashboard
   │
   ▼
Business Insights
   │
   ▼
Recommendations
```

---

## 16. Lessons Learned

*[To be completed at the end of the project]*

- Challenges encountered
- Limitations identified
- Improvements for future projects

---

## 17. Executive Summary

> *This section will be written at the very end of the project after all findings, recommendations, and conclusions are finalized.*

---

*Document Status: In Progress — The document will be updated continuously as each project phase is completed.*
