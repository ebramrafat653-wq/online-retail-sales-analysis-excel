# Power Query Logic Documentation

## Online Retail Sales Analysis – Power Query ETL

---

## 1. Purpose

This document describes the Power Query transformation logic used to prepare the Online Retail dataset for the Excel Data Model, Power Pivot measures, exploratory analysis, and executive dashboards.

The transformation process is designed to:

* load the raw CSV dataset;
* enforce consistent data types;
* remove exact duplicate records;
* classify transaction behavior;
* calculate transaction-level revenue;
* create reusable time dimensions;
* classify product versus non-product records;
* create an analytical classification used by the DAX measures.

The original source columns are preserved throughout the transformation process.

---

## 2. Source and Initial Loading

The query loads the raw CSV file from the project data source.

### Source configuration

* **Source:** `data/raw/online_retail.csv`
* **Delimiter:** comma
* **Encoding:** Windows-1252 (`1252`)
* **Expected columns:** 8

The source columns are promoted to headers before further transformations are applied.

---

## 3. Data Type Standardization

The following source data types are explicitly assigned:

| Column        | Power Query Type |
| ------------- | ---------------- |
| `InvoiceNo`   | Text             |
| `StockCode`   | Text             |
| `Description` | Text             |
| `Quantity`    | Int64            |
| `InvoiceDate` | Date/Time        |
| `UnitPrice`   | Number           |
| `CustomerID`  | Int64            |
| `Country`     | Text             |

This ensures that subsequent calculations, date transformations, and classification rules operate on consistent data types.

---

## 4. Exact Duplicate Removal

The query applies:

```m
Table.Distinct(#"Changed Type")
```

This removes exact duplicate rows across all columns.

Only exact duplicates are removed. No additional deduplication rule is applied based on `InvoiceNo`, `StockCode`, or other individual fields.

### Business rationale

The project treats identical duplicate rows as duplicate records rather than separate business transactions.

This approach minimizes unnecessary data loss while reducing the risk of duplicate-driven inflation in analytical metrics.

---

## 5. Transaction Classification

A `TransactionType` field is created using the following business rules:

```m
each
    if Text.StartsWith([InvoiceNo], "C")
    then "Cancellation"

    else if [Quantity] < 0
        and not Text.StartsWith([InvoiceNo], "C")
    then "Inventory Adjustment"

    else "Sale"
```

### Classification rules

| Rule                                                   | TransactionType        |
| ------------------------------------------------------ | ---------------------- |
| `InvoiceNo` begins with `C`                            | `Cancellation`         |
| `Quantity < 0` and `InvoiceNo` does not begin with `C` | `Inventory Adjustment` |
| All remaining records                                  | `Sale`                 |

### Important interpretation note

`Inventory Adjustment` is an **analytical classification derived from transaction patterns**. It should not be interpreted as an authoritative inventory-management record type unless supported by an external operational system.

The classification is based on the observable characteristics of the transaction data.

---

## 6. Revenue Calculation

A transaction-level `Revenue` field is created using:

```m
[Quantity] * [UnitPrice]
```

The resulting field is converted to `Currency.Type`.

### Definition

```text
Revenue = Quantity × UnitPrice
```

Because `Quantity` may be negative for cancellation or adjustment-related records, the resulting revenue can also be negative.

### Analytical implication

* Positive revenue generally represents positive-value transaction lines.
* Negative revenue represents negative transaction-line value resulting from negative quantities.
* Revenue is therefore a **signed transaction-line measure**.

The DAX model determines how these signed values are aggregated into the project's broader revenue measures.

---

## 7. Date Feature Engineering

Several reusable date fields are created from `InvoiceDate`.

### 7.1 Year

```m
Date.Year([InvoiceDate])
```

Purpose:

* annual filtering;
* time aggregation;
* reporting segmentation.

### 7.2 MonthNumber

```m
Date.Month([InvoiceDate])
```

Purpose:

* chronological month sorting;
* correct ordering of `MonthName`.

### 7.3 MonthName

```m
Date.MonthName([InvoiceDate])
```

Purpose:

* readable month labels in reports and PivotTables.

### 7.4 YearMonth

```m
Date.ToText(
    DateTime.Date([InvoiceDate]),
    "yyyy-MM"
)
```

Purpose:

* stable monthly reporting key;
* chronological month grouping;
* Month-over-Month analysis.

### 7.5 DayName

```m
Date.DayOfWeekName(
    DateTime.Date([InvoiceDate]),
    "en-US"
)
```

Purpose:

* weekday analysis;
* operational demand analysis.

### 7.6 WeekdayNumber

```m
Date.DayOfWeek(
    DateTime.Date([InvoiceDate]),
    Day.Monday
) + 1
```

Purpose:

* ordering weekdays from Monday through Sunday;
* preventing alphabetical weekday ordering in reports.

### 7.7 InvoiceDateOnly

```m
Date.From([InvoiceDate])
```

Purpose:

* daily-level analysis without time-of-day granularity.

---

## 8. Stock Code Pattern Classification

A `StockCodePattern` field is created based on the character structure of `StockCode`.

The query applies the following logic:

```m
each
    if Text.Select([StockCode], {"0".."9"}) = [StockCode]
    then "Numeric"

    else if Text.Select([StockCode], {"A".."Z","a".."z"}) = [StockCode]
    then "Text"

    else "Mixed"
```

### Resulting classifications

| StockCode Pattern | Meaning                                              |
| ----------------- | ---------------------------------------------------- |
| `Numeric`         | Stock code contains only numeric characters          |
| `Text`            | Stock code contains only alphabetic characters       |
| `Mixed`           | Stock code contains a combination of character types |

### Important interpretation note

This is a **structural classification rule**, not a formal product master classification.

---

## 9. ItemType Classification

An `ItemType` field is derived from `StockCodePattern`.

The logic is:

```m
each
    if [StockCodePattern] = "Text"
    then "Non-Product"
    else "Product"
```

### Resulting rule

| StockCodePattern | ItemType      |
| ---------------- | ------------- |
| `Text`           | `Non-Product` |
| `Numeric`        | `Product`     |
| `Mixed`          | `Product`     |

### Business interpretation

The project uses this rule as an analytical method for distinguishing product-oriented records from non-product records.

It should not be interpreted as a fully authoritative merchandise master classification.

---

## 10. AnalysisType Classification

The final analytical field is `AnalysisType`.

It is derived from both `TransactionType` and `ItemType`.

The implemented logic is:

```m
each
    if [TransactionType] = "Cancellation"
        and [ItemType] = "Product"
    then "Cancellation"

    else if [TransactionType] = "Sale"
        and [ItemType] = "Product"
    then "Sale"

    else "Non-Product"
```

### Final AnalysisType categories

| TransactionType      | ItemType    | AnalysisType   |
| -------------------- | ----------- | -------------- |
| Cancellation         | Product     | `Cancellation` |
| Sale                 | Product     | `Sale`         |
| Inventory Adjustment | Product     | `Non-Product`  |
| Inventory Adjustment | Non-Product | `Non-Product`  |
| Sale                 | Non-Product | `Non-Product`  |
| Cancellation         | Non-Product | `Non-Product`  |

### Important implementation caveat

Although `TransactionType` contains the category:

```text
Inventory Adjustment
```

the final `AnalysisType` field does **not** contain `Inventory Adjustment`.

Inventory-adjustment records are mapped to:

```text
Non-Product
```

when the final `AnalysisType` is generated.

This distinction is important because the DAX model uses `AnalysisType` as the primary analytical filtering layer.

---

## 11. Transformation Flow

The full transformation sequence can be summarized as:

```text
Raw CSV
   │
   ▼
Promote Headers
   │
   ▼
Set Source Data Types
   │
   ▼
Remove Exact Duplicates
   │
   ▼
Create TransactionType
   │
   ├── Cancellation
   ├── Inventory Adjustment
   └── Sale
   │
   ▼
Create Revenue
   │
   ▼
Create Date Features
   │
   ├── Year
   ├── MonthNumber
   ├── MonthName
   ├── YearMonth
   ├── DayName
   ├── WeekdayNumber
   └── InvoiceDateOnly
   │
   ▼
Create StockCodePattern
   │
   ├── Numeric
   ├── Text
   └── Mixed
   │
   ▼
Create ItemType
   │
   ├── Product
   └── Non-Product
   │
   ▼
Create AnalysisType
   │
   ├── Sale
   ├── Cancellation
   └── Non-Product
   │
   ▼
Power Pivot / DAX Data Model
```

---

## 12. Analytical Design Principles

The Power Query layer follows several analytical design principles:

### Preserve source information

Original transaction fields remain available after transformation.

### Separate raw classification from reporting classification

`TransactionType` captures the transaction behavior, while `AnalysisType` provides the simplified filtering layer used by analytical measures.

### Use explicit date dimensions

Dedicated date attributes simplify time-based aggregation and improve dashboard consistency.

### Preserve signed transaction values

Negative quantities and negative revenue are retained when present rather than being removed automatically.

### Minimize unsupported assumptions

Records are retained when the available evidence does not justify treating them as invalid.

---

## 13. Known Limitations of the ETL Logic

The current transformation logic has several documented limitations:

1. `ItemType` is inferred from the structural pattern of `StockCode`; it is not derived from an authoritative product master.
2. `Inventory Adjustment` is a transaction-level analytical classification rather than a verified operational inventory system classification.
3. `AnalysisType` maps Inventory Adjustment records to `Non-Product`.
4. `Revenue` is calculated at transaction-line level and may be negative when `Quantity` is negative.
5. The transformation logic does not independently establish the business root cause of cancellations or negative quantities.

These limitations are disclosed so downstream KPI calculations and business interpretations are not presented with greater certainty than the data supports.

---

## 14. Relationship to the DAX Model

The prepared table produced by Power Query becomes the source table for the Power Pivot / DAX model.

The main analytical fields used by downstream measures include:

* `Revenue`
* `TransactionType`
* `AnalysisType`
* `InvoiceNo`
* `CustomerID`
* `StockCode`
* `Country`
* `Year`
* `MonthNumber`
* `YearMonth`
* `DayName`
* `WeekdayNumber`

The DAX layer applies business metrics and aggregation logic after these transformations are completed.

---

## 15. Reproducibility Note

The transformation logic documented here reflects the current Power Query implementation used by the workbook.

The source path in the original workbook points to a local Windows directory. Because the path is machine-specific, the query is not directly portable to another environment without updating the source location.

For portfolio reproducibility, the raw dataset should remain stored in:

```text
data/raw/online_retail.csv
```

and the source path should be documented as a local configuration dependency rather than treated as a portable absolute path.

---

## 16. Summary

The Power Query layer provides the project's analytical preparation stage by:

* loading and typing the raw data;
* removing exact duplicates;
* classifying transaction behavior;
* calculating transaction-level revenue;
* engineering reusable time dimensions;
* profiling stock-code structure;
* separating product and non-product records;
* creating the final analytical `AnalysisType` used by the DAX model.

The transformation layer is intentionally conservative: it preserves transaction records unless the available evidence supports a clear transformation or classification rule.
