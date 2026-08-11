# 10. Data Dictionary

This data dictionary documents the original dataset columns (from `data/raw/online_retail.csv`) and the engineered analytical columns used in the Sales Performance Analysis project. Where possible derivation logic and data types are taken from the project documentation and the raw CSV. No Power Query M code was found in the repository; any queries embedded in the Excel workbook were not inspected. Where the repository lacks a serialized Power Query pipeline, derivation descriptions follow the project's documentation and the CSV structure (not inferred business rules).

## 10.1 Original Dataset Columns

The following table documents the eight original columns present in `data/raw/online_retail.csv`.

| Column | Data Type | Description | Business Meaning | Data Quality Notes |
|---|---:|---|---|---|
| InvoiceNo | Text | Invoice identifier as provided in the source CSV. May contain numeric-only values or prefixed values for cancelled transactions (e.g., values starting with "C"). | Identifies the invoice (transaction). When prefixed with "C" the invoice denotes a cancellation. | Source CSV contains numeric-like values (e.g. 536365). Treat as text because cancellation markers may be present. |
| StockCode | Text | Item / stock code as provided in the source CSV (alphanumeric values such as "85123A", "POST"). | Product or ledger code identifying the sold item or non-product line (postage, service). | Contains mixed alphanumeric values and explicit non-product codes (example: "POST"). |
| Description | Text | Product or line description (free text). | Human-readable product name or description for reporting and grouping. | Contains occasional missing/empty descriptions; rely on `StockCode` when `Description` is empty. |
| Quantity | Whole Number | Quantity value for the line as provided in the CSV. | Number of units in the transaction line. Negative values indicate returns or inventory adjustments depending on `InvoiceNo` and business rules. | Contains negative values (some correspond to invoices starting with "C", others represent inventory adjustments). Review required per analysis. |
| InvoiceDate | Date/Time | Date and time when the invoice was generated (Raw CSV format: MM/DD/YYYY HH:MM:SS AM/PM). | Transaction timestamp (used to derive Year, Month, Day and other time dimensions). | All values in the CSV follow the same timestamp format; timezone not provided. |
| UnitPrice | Decimal Number | Unit price as provided in the CSV. | Price charged per unit for the line item. | Contains zero-priced lines; no negative unit prices observed in the raw CSV sample. |
| CustomerID | Text | Customer identifier as provided in the CSV (numeric in source but may contain missing values). | Identifies the buyer; used for customer-level analysis when present. | Many records have missing `CustomerID` values. In the CSV values appear as floating numbers (e.g. 17850.0) — preserve original representation but treat as identifier (text) where appropriate. |
| Country | Text | Customer country as provided in the CSV. | Geographic dimension for revenue and country-level analysis. | Values are text country names; consistent across sample rows. |

## 10.2 Engineered Analytical Columns

Note: No external Power Query M scripts were found in the repository. The engineered column names and derivation logic documented here are taken from the project's documentation (`docs/Project Documentation.md`) and from inspecting the raw CSV. If an authoritative Power Query pipeline exists inside the Excel workbook, the workbook was not programmatically inspected; any differences between workbook queries and this file are noted in the Final Verification section.

| Column | Data Type | Derivation / Logic | Business Purpose | Notes | Examples |
|---|---:|---|---|---|---|
| TransactionType | Text (enumeration) | Derived from `InvoiceNo`, `Quantity`, and transaction business rules. Values documented in project materials: "Sale", "Cancellation", "Inventory Adjustment". | Classify transactions consistently for sales, cancellations/returns, and inventory adjustment analysis. | Exact M-code not present in repository; derivation follows project documentation. Confirm in workbook queries if available. | "Sale", "Cancellation", "Inventory Adjustment" |
| Revenue | Decimal Number | Formula: `[Quantity] * [UnitPrice]` | Transaction-level revenue used for aggregation, trend analysis and contribution calculations. | Standard numeric multiplication of existing columns. | 15.30, 0.00, -12.75 |
| Year | Whole Number | Extracted from `InvoiceDate` (calendar year). | Support yearly filtering and year-over-year comparisons. | Uses calendar year from `InvoiceDate`. | 2010, 2011 |
| MonthNumber | Whole Number | Extracted from `InvoiceDate` as 1–12. | Preserve chronological month ordering and act as a sort key for `MonthName`. | Supporting sort key; do not display as primary month label in reports. | 12, 1, 3 |
| MonthName | Text | Extracted from `InvoiceDate` as English month name (e.g., "January"). | Readable month label for reports, Pivot Tables, and dashboards. | Use `MonthNumber` to control chronological ordering. | "December", "January", "March" |
| YearMonth | Text | Formatted from `InvoiceDate` as `YYYY-MM` (example: "2011-03"). | Stable, locale-independent monthly analytical label used for trend analysis. | Primary monthly analytical label. | "2010-12", "2011-01" |
| DayName | Text | Extracted from `InvoiceDate` as English weekday name (Monday–Sunday). | Analyze sales and transaction patterns by day of week. | Use `WeekdayNumber` to control sort order. | "Wednesday", "Monday" |
| WeekdayNumber | Whole Number | Derived from `InvoiceDate` with Monday = 1 through Sunday = 7. | Sorting key for `DayName` so weekday reports appear in chronological order. | Supporting sort key. | 1, 7 |
| InvoiceDateOnly | Date | Derived from `InvoiceDate` by removing the time component (date portion only). | Support daily-level reporting and avoid time-of-day granularity when not required. | Useful for daily aggregates and joins at date granularity. | 2010-12-01, 2011-03-15 |
| StockCodePattern | Text (enumeration) | Described in project documentation as classification of `StockCode` structure (examples: "Numeric", "Mixed", "Text"). | Support product-code profiling and help distinguish standard product codes from special/non-product codes (e.g., postage). | Project documentation provides the intended categories; exact classification rules (M code) were not found in repository. | "Numeric" (10002), "Mixed" (85123A), "Text" (POST) |
| ItemType | Text (enumeration) | Derived from `StockCodePattern` and business classification rules; documented values include "Product" and "Non-Product". | Distinguish product items from service/fee/discount/non-product transaction lines. | Exact mapping rules should be verified in the workbook if queries exist. | "Product", "Non-Product" |
| AnalysisType | Text (enumeration) | Derived from `TransactionType` and `ItemType` to present an analysis-oriented category (e.g., product sales, product cancellations, non-product records). | Provide a consistent analytical layer for filtering and separating product sales, product cancellations, and non-product records during analysis. | Derivation described in project documentation; M code not available in repo. | "Sale", "Cancellation", "Non-Product" |

---

## Final Verification

- Files inspected: `data/raw/online_retail.csv`, `docs/Project Documentation.md`, `docs/data_dictionary.md` (this file), `README.md`, `excel/workbook/Sales_Performance_Analysis.xlsx` (not programmatically inspected). `data/processed/` is empty.
- Original columns documented: 8
- Engineered columns documented: 12
- Discrepancies / verification notes:
	- No Power Query M (.pq/.m) files or serialized query scripts were found in the repository. The Excel workbook `excel/workbook/Sales_Performance_Analysis.xlsx` may contain Power Query steps; those embedded queries were not programmatically inspected by this update. Consequently, derivation logic for classification fields (`TransactionType`, `StockCodePattern`, `ItemType`, `AnalysisType`) is documented from the project documentation rather than the authoritative M code.
	- Data types for original columns were determined from the source CSV and the project documentation. If the workbook contains explicit `Changed Type` steps that differ, the workbook should be inspected and this file updated to reflect exact `Changed Type` steps.

If you would like, I can attempt to extract Power Query M from the Excel workbook (this requires opening the workbook with Excel or a tool that can extract embedded queries). Otherwise this data dictionary reflects the authoritative CSV plus project documentation.

