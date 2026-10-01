# Cafe Sales Data Cleaning Portfolio Project (SQL)

## 📌 Project Overview
This repository features an end-to-end data cleaning project using **MySQL**. The objective was to take a messy, unstandardized transactional dataset from a fictional cafe environment (`dirty_cafe_sales`) and transform it into a business-ready staging layer (`dirty_cafe_sales_1`) optimized for reliable reporting.

## 🛠️ Skills & SQL Concepts Used
* **Staging Database Strategy:** Duplicating structures via `CREATE TABLE ... LIKE`.
* **Advanced Filtering & CTEs:** Isolating duplicate entries dynamically using window functions (`ROW_NUMBER() OVER PARTITION BY`).
* **Conditional Logic:** Rebuilding missing inventory tags using nested `CASE WHEN` workflows.
* **Regex Data Validation:** Standardizing numerical values with regular expressions (`REGEXP '[a-zA-Z]'`).
* **Schema Modifications & Type Casting:** Transforming raw text timestamps using `STR_TO_DATE` and modifying structure arrays via `ALTER TABLE`.

## 🧼 Cleaning Workflow

### 1. Duplicate Handling
Identified repeating transactions utilizing a **Common Table Expression (CTE)** paired with a partition system targeting key markers (`Transaction ID`, `Item`, `Quantity`, and `Price`).

### 2. Logical Data Imputation
Discovered that "unknown" items shared consistent pricing tiers with validated catalog lines. Leveraged a `CASE` map to clean out bad strings:
* `$1.50` ➡️ Tea
* `$2.00` ➡️ Coffee
* `$5.00` ➡️ Salad
* ...and more.

### 3. Financial Inconsistency Repair
Cleared textual error errors out of financial totals, replacing missing data dynamically via an algorithmic calculation (`Quantity * Price per unit`).

### 4. Text and Datetime Standardization
* Normalized missing parameters across `Payment Method` and `Location` rows.
* Converted varied string entries into uniform database metrics using native datetime parsing tools.

