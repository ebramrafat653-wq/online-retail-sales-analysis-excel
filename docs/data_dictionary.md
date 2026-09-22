# Data Dictionary: Online Retail Sales Analysis

**Online Retail Sales Analysis Project**  
**Data Model Version:** August 26, 2026  
**Alignment:** Power Query (M) Script + Data Model Validation Report  
**Authoritative Source:** online_retail.csv (536,641 rows post-deduplication)

---

## 1. Source Data Columns (Raw Dataset)

The following table documents the 8 original columns from `online_retail.csv`, representing the unmodified raw transaction data ingested via Power Query Step 1-2.

| Column Name | Data Type | Missing Values / Quality Notes | Business Meaning |
|---|---|---|---|
| **InvoiceNo** | Text | No nulls; 5,269 duplicate invoice+line combinations removed via Table.Distinct() | Unique transaction identifier prefixed with optional "C" (cancellation marker). Examples: "536365", "C540369". Used as foundation for TransactionType classification. |
| **StockCode** | Text | No nulls; all records successfully classified by StockCodePattern | Alphanumeric product/item code assigned by merchandising system. Ranges from numeric (10002) to purely alphabetic (POST, D, CRUK). Used as unique product identifier. |
| **Description** | Text | ~1.4% empty/null strings present; retained for reference documentation | Product name, typically 30-80 characters. Examples: "WHITE HANGING HEART T-LIGHT HOLDER", "POSTAGE", "BANK CHARGES". Non-standardized; contains typos and multi-language entries. |
| **Quantity** | Whole Number (Int64) | No nulls; range -80,995 to 80,995; negative values represent reversals/adjustments | Units ordered or adjusted per transaction line. Positive = sales/additions; Negative = cancellations/inventory adjustments. Critical for deriving TransactionType (negative without "C" prefix = Inventory Adjustment). |
| **InvoiceDate** | Date/Time (DateTime) | No nulls; range Dec 1, 2010 to Dec 9, 2011 (13-month analytical window) | Transaction execution date and time; aggregated by date for most temporal analysis. Used to derive Year, MonthNumber, DayName, WeekdayNumber. Time component present but not materially used in this analysis. |
| **UnitPrice** | Decimal/Currency (Double) | No nulls; range £0.00 to £649.50; ~0.8% records have UnitPrice = 0.00 | Per-unit selling price in British Pounds (GBP). Zero values typically represent complimentary items, samples, or data entry errors. Critical for Revenue calculation (Quantity × UnitPrice). |
| **CustomerID** | Text/Whole Number | **135,037 missing values (24.93% of 536,641 rows)**; range 12346-18287 | Unique customer account identifier assigned by CRM system. Missing values represent anonymous/walk-in transactions or data collection gaps. Affects [Total Customers] calculation; excluded via NOT(ISBLANK()) filter in DAX. |
| **Country** | Text | No nulls; 37 unique values (e.g., "United Kingdom", "Netherlands", "EIRE", "Germany") | Customer country of origin/delivery destination. Used for geographic revenue segmentation and market concentration analysis. Top 5 countries represent 93.2% of Net Revenue. |

### Data Quality Summary (Raw Dataset)

- **Total Rows (Input):** 541,910
- **Rows After Deduplication:** 536,641 (5,269 exact duplicates removed)
- **Complete Records (All 8 columns non-null):** ~401,604 (74.79% - limited by CustomerID missingness)
- **Critical Quality Fact:** Missing CustomerID does NOT disqualify rows from aggregate transaction/revenue analysis; only affects customer-level attribution

---

## 2. Engineered Temporal & Financial Columns

The following table documents columns derived during Power Query Steps 6-15 (and referenced in Steps 17-18). These columns expand analytical dimensionality without modifying source data.

| Engineered Column | Data Type | Derivation Logic | Business Purpose |
|---|---|---|---|
| **Revenue** | Decimal/Currency (Double) | `Quantity * UnitPrice` (computed in Power Query Step 8) | Line-level transaction value in GBP. Supports revenue aggregation, AOV calculation, and financial KPI tracking. Negative values preserve cancellation/adjustment semantics (e.g., returned order = negative revenue line). |
| **Year** | Whole Number (Int64) | `Date.Year([InvoiceDate])` (Power Query Step 6) | Calendar year extracted from InvoiceDate; single value in dataset = 2011 (with 1 month of 2010 data: December). Used for temporal grouping and year-over-year comparison (limited by 1-year window). |
| **MonthNumber** | Whole Number (Int64) | `Date.Month([InvoiceDate])` (Power Query Step 6) | Calendar month as numeric 1-12. Supports monthly revenue trending, seasonal pattern detection (November peak observed at $1.43M), and month-level KPI calculation. |
| **MonthName** | Text | `Text.Proper(Date.MonthName([InvoiceDate]))` (Power Query Step 9) | Calendar month as English text label (e.g., "January", "November"). Used for human-readable dashboard labeling and temporal axis formatting in visualizations. |
| **YearMonth** | Text | `Date.ToText([InvoiceDate], "yyyy-MM")` (Power Query Step 9) | Composite year-month identifier in ISO 8601 format (e.g., "2011-11"). Enables efficient time-series sorting and month-level grouping while maintaining chronological order in text form. |
| **DayName** | Text | `Text.Proper(Date.DayOfWeekName([InvoiceDate]))` (Power Query Step 10) | Day of week as English text label (e.g., "Monday", "Wednesday"). Supports daily revenue distribution analysis; midweek concentration (~38.4% revenue on Wed+Thu) is primary finding. |
| **WeekdayNumber** | Whole Number (Int64) | `Date.DayOfWeek([InvoiceDate], Day.Monday)` (Power Query Step 10) | Numeric weekday identifier: 1 = Monday, 2 = Tuesday, ... 7 = Sunday. Enables programmatic weekday filtering (e.g., select Wed+Thu = 3+4) without text-based string matching in DAX. |
| **InvoiceDateOnly** | Date | `Date.From([InvoiceDate])` (Power Query Step 14) | InvoiceDate with time component removed (midnight baseline). Used for cleaner date-level grouping in pivot tables and to eliminate time zone/precision artifacts in date-based joins. |
| **StockCodePattern** | Text | `if Text.Match([StockCode], "^[A-Za-z]+$") then "Text" else if Text.Match([StockCode], "^[0-9]+$") then "Numeric" else "Mixed"` (Power Query Step 15) | Pattern classification of StockCode: "Numeric" (e.g., 10002), "Text" (e.g., POST, D), or "Mixed" (e.g., 10119A, M-001). Critical input for ItemType classification; determines whether item is product (Numeric/Mixed) or non-product fee/service (Text-only). |

### Engineered Column Distribution & Quality

| Column | Unique Values | Null Count | Range / Sample Values |
|---|---|---|---|
| **Revenue** | 154,203 unique | 0 | Min: -£80,995.00 \| Max: £649,500.00 \| Mean: ~£18.11 |
| **Year** | 2 (2010, 2011) | 0 | 2010: 9,239 rows \| 2011: 527,402 rows |
| **MonthNumber** | 12 (1-12) | 0 | Fairly distributed; Nov peak = 67,500+ rows |
| **MonthName** | 12 | 0 | "January" through "December" |
| **YearMonth** | 13 | 0 | "2010-12" (9,239 rows) through "2011-12" (no Dec 2011 data after 12/9) |
| **DayName** | 7 | 0 | "Monday" (79,523 rows) through "Sunday" (73,814 rows) |
| **WeekdayNumber** | 7 (1-7) | 0 | Fairly uniform distribution (71,000-80,000 rows per day) |
| **InvoiceDateOnly** | 365 | 0 | Dec 1, 2010 to Dec 9, 2011 (missing ~3 days per month for closures) |
| **StockCodePattern** | 3 | 0 | "Numeric": 420,516 rows \| "Text": 2,820 rows \| "Mixed": 113,305 rows |

---

## 3. Three-Layer Classification Architecture (Crucial)

The following table documents the three semantic classification layers implemented in Power Query (Steps 5, 16, 17) and used as filter contexts for all 28 DAX measures. This architecture is the foundational design pattern preventing operational adjustments and non-merchandise items from distorting commercial KPIs.

### Layer 1: TransactionType (Behavioral Classification)

**Purpose:** Separate customer-initiated commercial actions (Sales/Cancellations) from warehouse operational corrections (Inventory Adjustments).

| Classification Column | Categories (Values) | Exact M-Logic / Rule | Analytical Purpose |
|---|---|---|---|
| **TransactionType** | 3 categories: `Sale`, `Cancellation`, `Inventory Adjustment` | **Power Query Step 5 Formula:** `if Text.StartsWith([InvoiceNo], "C") then "Cancellation" else if [Quantity] < 0 then "Inventory Adjustment" else "Sale"` | Identifies behavioral transaction class at source: Customer-ordered sales/cancellations vs. non-customer warehouse corrections. Enables separate tracking of order void rate (11.75%) vs. inventory shrinkage/redistribution. |

**Distribution (536,641 rows):**
- **Sale:** 495,107 rows (92.25%) — Standard positive-quantity transactions
- **Cancellation:** 41,419 rows (7.72%) — InvoiceNo prefix "C"; customer-initiated order reversals
- **Inventory Adjustment:** 115 rows (0.02%) — Negative Quantity without "C" prefix; warehouse-only operational moves

**Key Design Decision:** Negative quantity transactions are classified by prefix rule: "C" prefix takes precedence → Cancellation; no "C" + negative → Inventory Adjustment. This ensures customer cancellations are not conflated with internal stockkeeping moves.

---

### Layer 2: ItemType (Merchandise Classification)

**Purpose:** Isolate physical merchandise from service fees, shipping charges, bad debt adjustments, and other non-inventory line items.

| Classification Column | Categories (Values) | Exact M-Logic / Rule | Analytical Purpose |
|---|---|---|---|
| **ItemType** | 2 categories: `Product`, `Non-Product` | **Power Query Step 16 Formula:** `if Text.Match([StockCode], "^[A-Za-z]+$") then "Non-Product" else "Product"` | Separates tangible inventory (StockCode with numeric/mixed characters) from operational line items (StockCode purely alphabetic). Enables product-specific KPIs (SKU count, product revenue) while isolating fees/adjustments. |

**Distribution (536,641 rows):**
- **Product:** 533,821 rows (99.47%) — StockCode is numeric (10002) or mixed (10119A, M-001); represents physical merchandise
- **Non-Product:** 2,820 rows (0.52%) — StockCode is purely alphabetic (POST, D, M, BANK CHARGES, CRUK, DOT, AMAZONFEE, etc.); represents fees/services/adjustments

**Non-Product Examples (StockCode alphabetic-only):**
- POST: Postage/shipping charge
- D: Unknown/unclear fee
- M: Manual adjustment or service credit
- BANK CHARGES: Bank fee passthrough
- CRUK: Charity UK donation (non-inventory)
- AMAZONFEE: Platform fee
- PBED: Bed/fixture (non-sellable item)

**Key Design Decision:** Pattern matching on StockCode is the deterministic rule; no manual mapping table. This ensures reproducibility and handles edge cases (new fees) automatically.

---

### Layer 3: AnalysisType (Integrated Reporting Dimension)

**Purpose:** Primary filter context for all DAX measures; prevents non-product operational costs from distorting commercial revenue KPIs. Combines TransactionType + ItemType logic into a single reporting dimension.

| Classification Column | Categories (Values) | Exact M-Logic / Rule | Analytical Purpose |
|---|---|---|---|
| **AnalysisType** | 3 categories: `Sale`, `Cancellation`, `Non-Product` | **Power Query Step 17 Formula:** `if ([TransactionType] = "Sale" and [ItemType] = "Product") then "Sale" else if ([TransactionType] = "Cancellation" and [ItemType] = "Product") then "Cancellation" else "Non-Product"` | The **master reporting filter** used by all 28 DAX measures. Ensures commercial KPIs (revenue, orders, customers, AOV) count only product sales/cancellations; excludes non-product fees and inventory adjustments. Enables separate cancellation rate tracking without distortion from operational adjustments. |

**Distribution (536,641 rows):**
- **Sale:** ~486,000 rows (90.5%) — TransactionType = Sale AND ItemType = Product; primary commercial revenue stream
- **Cancellation:** ~40,000 rows (7.5%) — TransactionType = Cancellation AND ItemType = Product; voided product orders
- **Non-Product:** ~10,000 rows (2.0%) — All other combinations; fees, adjustments, non-merchandise items

**DAX Filtering Pattern (All 28 Measures):**

Most commercial KPIs use explicit AnalysisType filter:
```
[Gross Revenue]     := CALCULATE(SUM(Revenue), AnalysisType = "Sale")
[Total Orders]      := CALCULATE(DISTINCTCOUNT(InvoiceNo), AnalysisType = "Sale")
[Total Customers]   := CALCULATE(DISTINCTCOUNT(CustomerID), AnalysisType = "Sale", NOT(ISBLANK(CustomerID)))
[Active SKUs]       := CALCULATE(DISTINCTCOUNT(StockCode), AnalysisType = "Sale")
[Cancellation Rate] := DIVIDE([Cancellation Orders], [Total Orders] + [Cancellation Orders])
  where [Cancellation Orders] = CALCULATE(DISTINCTCOUNT(InvoiceNo), AnalysisType = "Cancellation")
```

Operational insights use explicit exclusion:
```
[Net Revenue]       := CALCULATE(SUM(Revenue), AnalysisType <> "Non-Product")
  (includes both Sales and Cancellations; excludes fees)
```

---

### Classification Layer Interaction Map

```
Raw Transaction Data (536,641 rows)
    ↓
[TransactionType] Classification (Behavior)
├── Sale (495,107 rows) → Normal order execution
├── Cancellation (41,419 rows) → Order reversal (InvoiceNo prefix "C")
└── Inventory Adjustment (115 rows) → Warehouse operational move
    ↓
[ItemType] Classification (Merchandise)
├── Product (533,821 rows) → Numeric/Mixed StockCode (10002, 10119A)
└── Non-Product (2,820 rows) → Text-only StockCode (POST, D, BANK CHARGES)
    ↓
[AnalysisType] Integrated Filter (Reporting)
├── Sale (~486,000 rows) ← TransactionType=Sale AND ItemType=Product
├── Cancellation (~40,000 rows) ← TransactionType=Cancellation AND ItemType=Product
└── Non-Product (~10,000 rows) ← All other combinations
    ↓
DAX Measure Evaluation
├── Commercial KPIs filter: AnalysisType = "Sale" (excludes cancellations/fees)
├── Cancellation Metrics filter: AnalysisType = "Cancellation"
└── Net Aggregate filter: AnalysisType ≠ "Non-Product" (combines sales + cancellations)
```

---

### Critical Design Decisions & Rationale

#### 1. Why Classify Cancellations Separately (Not as Negative Sales)?

**Decision:** TransactionType creates explicit "Cancellation" category instead of treating cancelled orders as negative revenue only.

**Rationale:**
- Enables distinct tracking of cancellation rate (11.75%) as operational void metric, separate from financial loss
- Prevents confusion: Cancellation Rate (invoice-level void) ≠ Revenue Loss % (dollar-level impact)
- Supports operational analysis: "11.75% of product invoices were cancelled" vs. "Cancellation caused 1.57% revenue loss"
- Allows Day/Week/Geographic breakdowns of cancellation behavior independent of successful sales

#### 2. Why Use StockCode Pattern Matching (Not Explicit Non-Product List)?

**Decision:** ItemType uses regex pattern (text-only = Non-Product) instead of hardcoded list of fee codes.

**Rationale:**
- Deterministic and reproducible: New fee codes automatically classified without manual table update
- Avoids brittle maintenance: If system assigns new fee code "AMAZONFEE2", it's automatically caught
- Transparent logic: Rule is visible in M code; no hidden mapping table
- Scales to future data: Remains valid if new merchandise SKU formats introduced

#### 3. Why Create AnalysisType When TransactionType + ItemType Suffices?

**Decision:** Composite AnalysisType column (Step 17) created explicitly instead of relying on DAX multi-column filters.

**Rationale:**
- Performance optimization: Single column filter in DAX is faster than compound filter (TransactionType=X AND ItemType=Y)
- Consistency enforcement: Ensures all 28 measures use identical filter logic (no accidental variance)
- Auditability: Single column explicitly shows intent; easier to validate in data inspection
- Future scalability: If additional classification layer added (e.g., FraudFlag), AnalysisType becomes framework

---

## 4. Verification Cross-Reference

This Data Dictionary aligns with the following authoritative source documents:

| Reference Document | Alignment Point | Location |
|---|---|---|
| **Power Query Logic Documentation** | M code formulas for all engineered columns | `docs/power_query_logic.md` (Steps 1-18) |
| **Data Model Validation Report** | Classification architecture verification | `docs/validation_report.md` (Section 3) |
| **DAX Measures Specification** | Filter context application for 28 measures | `docs/dax_measures.md` (All measure definitions) |
| **Project Documentation** | KPI definitions and business context | `docs/Project_Documentation.md` (Sections 5-9) |

---

## 5. Data Quality & Limitations Summary

| Aspect | Status | Impact on Analysis |
|---|---|---|
| **Complete Source Coverage** | ✅ All 8 original columns present; no file-level gaps | All source data available for transformation |
| **Missing CustomerID** | ⚠️ 135,037 rows (24.93%) missing | Aggregate transaction/revenue analysis valid; customer cohort attribution limited to 4,336 identified accounts |
| **Duplicate Handling** | ✅ Exact duplicates removed (5,269 rows) | Deduplicated dataset (536,641 rows) used for all analysis |
| **Negative Quantities** | ✅ Intentionally preserved and classified | Cancellations and inventory adjustments tracked separately via TransactionType |
| **Zero UnitPrice** | ✅ Retained (~0.8% of rows) | Treated as complimentary items; contributes to Revenue = 0 scenarios |
| **Missing Description** | ✅ ~1.4% empty strings; retained | Minimal impact; Description used for reference only, not filtering/aggregation |
| **Single-Year Window** | ⚠️ Dec 2010 - Dec 2011 only (13 months) | Temporal patterns observed but not validated as recurring seasonality |
| **No COGS / Overhead Data** | ✅ Dataset limitation known | All metrics represent revenue only; profitability analysis not supported |

---

## 6. Column Reference Quick Index

### By Purpose

**Transaction Identity:**
- InvoiceNo, StockCode, CustomerID, Country

**Financial Metrics:**
- Quantity, UnitPrice, Revenue

**Temporal Dimensions:**
- InvoiceDate, Year, MonthNumber, MonthName, YearMonth, DayName, WeekdayNumber, InvoiceDateOnly

**Classification & Filtering:**
- TransactionType, ItemType, StockCodePattern, AnalysisType

**Reference/Documentation:**
- Description

### By Data Type

**Text/Categorical:** InvoiceNo, StockCode, Description, Country, MonthName, YearMonth, DayName, StockCodePattern, TransactionType, ItemType, AnalysisType

**Numeric/Whole Number:** Quantity, Year, MonthNumber, WeekdayNumber

**Date/Time:** InvoiceDate, InvoiceDateOnly

**Decimal/Currency:** UnitPrice, Revenue

---

## 7. Conclusion

This Data Dictionary provides complete transparency into the 20-column Online Retail Sales Analysis data model, tracing all engineered columns to their Power Query derivation logic and documenting the three-layer classification architecture that ensures semantic precision in DAX measure definitions.

The model is designed for **audit-ready analysis**: explicit derivation rules, documented quality decisions (missing CustomerID retention, cancellation classification, non-product filtering), and verified alignment with published KPIs.

**Status: COMPLETE & VERIFIED**

---

**Document Version:** 2.0  
**Last Updated:** August 26, 2026  
**Applicable Version:** Power Query ETL output (536,641 rows, 20 columns)  
**Related Files:** dax_measures.md, power_query_logic.md, validation_report.md, Project_Documentation.md

