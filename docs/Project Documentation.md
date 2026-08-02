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

*[Each cleaning step will be documented during execution]*

---

## 9. KPI Definitions

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
