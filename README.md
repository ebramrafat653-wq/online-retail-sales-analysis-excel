# 📊 Online Retail Sales Analysis – Excel BI Dashboard

## 🎯 Project Objective

Transform **541,910** raw transactional records into an auditable, interactive Business Intelligence (BI) model using Excel's modern data stack. This project uncovers geographic revenue concentrations, seasonal operational patterns, and cancellation baselines while delivering an executive-ready dashboard for data-driven decision-making.

---

## 🛠️ Tech Stack & Architecture

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Data Engineering** | Power Query (M) | ETL pipeline with exact deduplication, 3-layer semantic classification (TransactionType → ItemType → AnalysisType) |
| **Data Modeling** | Power Pivot & DAX | 28 verified measures with explicit filter contexts for consistent KPI reporting |
| **Visualization** | Excel Dashboard | Dynamic `Executive_Summary_00` with synchronized slicers (Country, Year, TransactionType) |

---

## 📈 Key Verified Metrics

| Metric | Value |
|--------|------:|
| **Net Revenue** | **$9,771,519.35** |
| **Total Recorded Invoices** | **25,900** |
| **Identified Transacting Customers** | **4,336** |
| **Active Product SKUs** | **3,820** |
| **Average Order Value (AOV)** | **$395.94** |
| **Baseline Cancellation Rate** | **11.75%** |

---

## 💡 Executive Insights

### 1️⃣ Geographic Concentration
**United Kingdom** generates **$8.28M** (~84.7% of Net Revenue). The **top 5 markets** (UK, Netherlands, EIRE, Germany, France) account for **93.2%** of total revenue, highlighting a concentrated commercial footprint.

### 2️⃣ Q4 / November Seasonality
Revenue peaked sharply in **November** at **$1.43M** across **3,462 orders**, establishing a clear seasonal operating narrative for commercial planning.

### 3️⃣ Midweek Demand Velocity
**Wednesday and Thursday** combined account for **~38.4%** of weekly order volume, establishing the primary weekly operational baseline for staffing and fulfillment planning.

---

## 📂 Repository Structure

```text
online-retail-sales-analysis-excel/
│
├── data/
│   ├── raw/                     # Original Kaggle dataset (541,910 rows)
│   └── processed/               # Cleaned dataset reference (536,641 rows)
│
├── excel/
│   └── workbooks/
│       └── Sales_Performance_Analysis.xlsx   # Final BI workbook (Power Pivot + Dashboard)
│
├── docs/                        # Complete project documentation
│   ├── Project_Documentation.md             # 18-section comprehensive guide
│   ├── data_dictionary.md                   # Column & engineered feature dictionary
│   ├── dax_measures.md                      # 28 DAX measures with definitions & caveats
│   ├── power_query_logic.md                 # M code logic & transformation flow
│   └── validation_report.md                 # Data validation & semantic audit
│
├── images/                      # Dashboard & report screenshots
├── reports/figures/             # Additional analytical figures
└── README.md                    # This file


---

## 🔗 Deep Dive Documentation

| Document | Description |
|----------|-------------|
| **[Full Project Documentation](docs/Project_Documentation.md)** | Complete 18-section guide: business context, data quality, EDA, recommendations |
| **[Data Validation & Audit Report](docs/validation_report.md)** | Technical verification of metrics, semantic audit, and integrity checks |
| **[DAX Measures Catalog](docs/dax_measures.md)** | Complete documentation of all 28 measures with formulas, definitions, and caveats |
| **[Data Dictionary](docs/data_dictionary.md)** | Column & engineered feature dictionary with distributions |
| **[Power Query Logic](docs/power_query_logic.md)** | Step-by-step ETL transformation documentation |

---

## 🔍 3-Layer Classification Architecture

The project implements a **three-layer semantic classification** to isolate commercial metrics from operational adjustments:

Raw Transaction (536,641 rows)
↓
Layer 1: TransactionType (Behavior)
├── Sale (92.25%)
├── Cancellation (7.72%)
└── Inventory Adjustment (0.02%)
↓
Layer 2: ItemType (Merchandise)
├── Product (99.47%)
└── Non-Product (0.52%)
↓
Layer 3: AnalysisType (Reporting)
├── Sale → Commercial KPIs
├── Cancellation → Cancellation Metrics
└── Non-Product → Excluded from Commercial Metrics
↓
DAX Measures (28 verified measures)

---

**Key Design Decision:** All 28 DAX measures explicitly filter on `AnalysisType` to exclude fees, services, and inventory adjustments from product-level KPIs.

---

## 📊 Verified Executive Metrics Breakdown

| Component | Calculation | Value |
|-----------|-------------|------:|
| **Gross Revenue (Sales)** | SUM(Revenue) WHERE AnalysisType = "Sale" | ~$9,930,000+ |
| **Cancellation Revenue** | SUM(Revenue) WHERE AnalysisType = "Cancellation" | ~($157,000) |
| **Net Revenue** | Gross Revenue + Cancellation Revenue | **$9,771,519.35** |
| **Total Recorded Invoices** | DISTINCTCOUNT(InvoiceNo) | **25,900** |
| **Total Orders (Sales)** | DISTINCTCOUNT(InvoiceNo) WHERE AnalysisType = "Sale" | **22,041** |
| **Total Cancellation Orders** | DISTINCTCOUNT(InvoiceNo) WHERE AnalysisType = "Cancellation" | **2,951** |
| **Baseline Cancellation Rate** | 2,951 / (22,041 + 2,951) | **11.75%** |

---

## 🧠 Technical Implementation Highlights

- **Power Query (M):** 18 transformation steps loading, typing, deduplicating, and classifying data
- **Power Pivot / DAX:** 28 measures with explicit filter contexts; no circular dependencies
- **Excel Dashboard:** Dynamic `Executive_Summary_00` with KPI cards + Country/Year/TransactionType slicers
- **Semantic Audit:** Full verification report with status tags (VERIFIED / UNVERIFIED / IMPLEMENTATION LIMITATION)

---

## 📝 Data Quality Notes

| Issue | Count | Handling |
|-------|------:|----------|
| Missing CustomerID | 135,037 (24.93%) | Retained for transaction analysis; excluded from customer count |
| Missing Description | 1,454 | Retained; reference only |
| Zero UnitPrice | 1,336 | Retained; treated as legitimate variation |
| Exact Duplicates Removed | 5,269 | Removed via Table.Distinct() |

---

## ⚠️ Data Limitations

This dataset spans a **single 13-month window** (December 2010 – December 2011) and contains **no cost data (COGS)** — all financial metrics represent realized revenue only. While the analysis describes observed patterns with high confidence, it does **not** support causal inference or multi-year forecasting without additional data.

---

## 📄 License

This project is created for **portfolio and educational purposes**. The dataset remains the property of its original provider and is used in accordance with applicable source terms.

---

**Status:** ✅ Ready for Portfolio Publication

---

*Built with  using Excel, Power Query, and DAX* | *Last Updated: August 27, 2026*