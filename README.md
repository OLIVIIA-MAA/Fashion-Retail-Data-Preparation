# Retail Sales Data Cleaning and Preparation

## Project Overview

This project prepares sales, product and inventory data from a real fashion retail company for a separate business analysis. The datasets were made available for legal use in this portfolio and contain actual transaction, product and warehouse records rather than generated examples.

The three tables are connected through product_id, but they represent different levels of detail. Sales Orders contains one record per order, Products contains one record per product, and Inventory contains one record per product and warehouse country. Before these tables could be used together, I needed to confirm their keys, identify records that could distort joins or calculations, and separate structural errors from values that were incomplete but still useful.

The notebook does not produce one table in which every field is treated as equally reliable. Records with invalid or conflicting identities are rejected, while records with reliable keys but incomplete, unusual or unresolved fields remain in the cleaned outputs with status columns or review flags. Smaller analytical subsets apply stricter requirements for specific uses without removing valid information from the broader tables.

## Data Quality Questions

The cleaning process was designed around several questions that affect the reliability of later analysis:

* What does one row represent in each table, and which fields can be used as keys?
* Are repeated rows technical duplicates or valid records at a different level of detail?
* Which categorical variants can be standardized through confirmed mappings?
* How can mixed and ambiguous dates be parsed without inventing information?
* Which records must be rejected, and which should remain with a review flag?
* Can the tables be joined without losing records or multiplying results?
* Can every source row be accounted for after cleaning and export?

These questions determined the cleaning rules and the conditions used to create the final analytical subsets.

## Dataset Overview and Key Results

| Dataset      | Source rows | Source columns | Level of detail                      | Provisional key                | Main analytical use                                             |
| ------------ | ----------: | -------------: | ------------------------------------ | ------------------------------ | --------------------------------------------------------------- |
| Sales Orders |     260,780 |              9 | One row per order                    | order_id                       | Order counts, sales values, markets, products and time analysis |
| Products     |       2,500 |              5 | One row per product                  | product_id                     | Product attributes, base prices and launch dates                |
| Inventory    |       3,741 |              4 | One product in one warehouse country | product_id + warehouse_country | Stock availability and update-date review                       |

## Cleaning Approach

### Source protection and schema control

All source columns were initially loaded as text, with pandas' automatic missing-value recognition disabled. This prevented identifiers, mixed date formats and text values resembling missing-data markers from being converted before inspection. After loading, text fields were normalized by removing leading and trailing spaces, reducing repeated internal whitespace and converting empty strings to missing values.

I then added the _source_row column to each dataset. It stores the original row number from the source file and remains in the cleaned, rejected and analytical outputs. This makes it possible to trace every record back to its original position and verify that no rows were lost, duplicated or added during processing.

Before cleaning the values, I compared the structure of each file with the expected schema. The validation checked whether all required columns were present, whether the files contained additional columns not defined in the expected structure, whether any column names were duplicated and whether the columns appeared in the correct order. All three datasets passed these checks. Their structures matched the expected schemas, with no missing, additional or duplicated columns.

The initial quality assessment was completed before any records were rejected. It identified 780 exact duplicate copies in Sales Orders, 629 empty order dates, 31,322 empty discount values, 91 empty product launch dates and 6 empty inventory update dates. I also checked the datasets for placeholder values such as NA, N/A, null, none and nan. None were found, so no additional rules were required to convert text placeholders into missing values.

### Preserving original values

Before transforming a source field, the notebook creates a corresponding _raw column. The raw and cleaned forms can therefore be compared in the same output. This is useful when reviewing standardized categories, parsed dates and numeric values that remain unresolved.

The raw columns are not substitutes for the original files. Their purpose is to preserve the evidence behind a cleaning decision and make flagged records easier to review during later analysis.

### Categorical standardization

Known spelling, capitalization, language and abbreviation variants are standardized through explicit dictionaries. The mappings cover product categories and subcategories, sales countries, order statuses and warehouse countries.

I did not use fuzzy matching. Similar text does not confirm that two values have the same business meaning, and an approximate match could silently place a record in the wrong market or product group. A value not covered by a confirmed dictionary remains visible and receives a missing or unrecognized flag.

Sales-country variants are mapped into ten markets. Order statuses are consolidated into COMPLETED, SHIPPED and CANCELLED, while warehouse countries are standardized to Czech Republic, Germany and Poland. After mapping, the notebook checks the remaining values against the expected sets rather than assuming that every transformation succeeded.

### Date parsing

The source files use complete ISO dates written as YYYY-MM-DD, complete day-first dates written as DD-MM-YYYY, and partial monthly values written as YYYY-MM. Dots and slashes are standardized to hyphens before format detection.

Partial monthly values are not assigned an invented day. They receive the status partial_month. Empty values receive missing, while impossible or unsupported dates receive invalid. Only confirmed complete dates are stored as parsed dates.

A day-month date is textually ambiguous when both components are between 1 and 12. The date format was identified using unambiguous examples from the same column. For instance, a value such as 23-06-2024 can only follow the DD-MM-YYYY format. Based on this pattern, ambiguous values such as 05-06-2024 were also interpreted as day-first dates. These records were retained with an ambiguity flag to show that their original format was not fully explicit.

### Rejection and analytical filtering

Exact duplicate copies are handled before key validation. The first occurrence remains in the working table, while later copies move to the relevant rejected table with rejection_reason = exact_duplicate_copy.

The remaining key rules reflect the level of detail in each dataset. Products requires a valid and unique product_id. Sales Orders requires valid order_id, customer_id and product_id values, and order_id must remain unique. Inventory requires a valid product_id, a recognized warehouse country and a unique combination of product_id and warehouse_country.

A problem in a non-key field does not automatically invalidate the entire record. Missing dates, zero values, missing discounts, stale stock updates and unusual quantities are retained when the remaining fields are still usable. These records receive statuses or flags and may be excluded from a specific analytical subset later.

## Data Quality Findings and Decisions

### Products

Products is the reference table used to add categories, subcategories, base prices and launch information to Sales Orders and Inventory. Each product ID must appear only once because a duplicated product record could multiply sales values when the tables are merged.

The table contained no exact duplicate copies. All 2,500 product IDs were present, consisted only of digits and remained unique after normalization. No product records were rejected. The identifiers were stored as text because they are used to match records between tables rather than perform calculations.

All category and subcategory values were present and recognized. No labels required standardization in the current file. All base prices were converted successfully to numeric values, with no missing, invalid, zero or negative prices.

Launch dates required separate treatment because an incomplete date does not make the entire product record unusable. The source contained 91 empty launch dates. After parsing, the table included 91 missing dates, 3 values containing only a year and month, and 10 invalid dates.

Another 148 values were ambiguous because both the day and month were between 1 and 12. They were interpreted as DD-MM-YYYY because the same column contained dates such as 23-06-2024, which could only follow the day-first format. These records retained an ambiguity flag to show that their original notation was not fully explicit.

Products with missing, partial or invalid launch dates remained in the cleaned table because their IDs, categories, subcategories and prices were still usable. I did not estimate the missing dates because the source provided no basis for doing so. The cleaned output keeps the original value, parsed date, detected format and date status. Analyses based on product launches should use only records with launch_date_status = valid.

### Sales Orders: Duplicate and Key Validation

Sales Orders contained 260,780 source rows, including **780 records that were exact copies across all nine source columns**. The repeated copies were moved to **sales_orders_rejected.csv**, while the first occurrence remained in the cleaned table.

After duplicate removal, **260,000 rows** remained. No order, customer or product ID was missing or invalid. There were also no conflicting or repeated order IDs. This confirmed that one row represents one complete order rather than a single item within a multi-row order. No additional aggregation was required.

Orders with valid identifiers remained in the cleaned table even when another field, such as the date or discount, was incomplete. These records could still support analyses that did not depend on the affected field.

### Sales Orders: Dates and Categories

The cleaned Sales Orders table contains 218,047 valid dates in YYYY-MM-DD format and 39,367 valid dates in DD-MM-YYYY format. It also contains 1,263 invalid dates, 694 values containing only a year and month, and 629 missing dates.

The column contained 23,798 dates that clearly followed the day-first format. For example, a value such as 23-06-2024 could only mean 23 June 2024. Based on this pattern, 15,569 values such as 05-06-2024 were also interpreted as DD-MM-YYYY. They were retained with an ambiguity flag because their original notation could not be interpreted with complete certainty on its own.

Missing, partial and invalid dates remain in **sales_orders_clean.csv** because the orders can still be identified and used in analyses that do not depend on time. They are excluded from **sales_orders_core_analysis.csv**, where a complete date is required for monthly, annual and seasonal reporting.

Country and order-status values were standardized using predefined mappings. The process changed 188,960 country entries and 220,033 status entries. After validation, no country or status remained missing or unrecognized.

Cancelled orders were retained because cancellation is a valid business event rather than a data error. These records can be used to analyse cancellation rates, but they should not automatically be included in revenue calculations.

### Sales Orders: Numeric Values and Discounts

Quantities were converted to an integer format that allows missing values to remain visible. No invalid, fractional or negative quantities were found, but 1,034 orders contained a quantity of zero.

Unit prices were also converted to a numeric format that supports missing values. No invalid or negative prices were found, but 1,608 orders contained a zero unit price.

The meaning of the zero values was not documented in the source files, so they were not changed or removed automatically. These rows remain in the broad cleaned table with review flags but are excluded from the core sales subset. gross_order_value is calculated only when both quantity and unit price are positive.

After duplicate removal, **31,243 discounts were missing**. No recorded discount had an invalid format or fell outside the range from 0 to 100.

Missing discounts remained null. A recorded value of zero confirms that no discount was applied, while a missing value means that the discount is unknown. Replacing missing values with zero would incorrectly classify unconfirmed orders as full-price transactions. Gross value can still be calculated, but net_order_value remains missing when the discount is unknown.

### Sales Orders: Quantity 608

The median positive order quantity is 3. The 95th percentile is 6, the 99th percentile is 8, and the maximum is 608.

Quantity 608 appears in **1,563 orders**. By comparison, quantities 9, 10 and 11 appear in 408, 100 and 25 orders. This shows that 608 is a repeated pattern rather than a single extreme observation. However, the available data does not confirm whether it is a valid business value or a data-quality problem.

I retained these records and created quantity_608_review_flag. They represent only **0.60% of structurally cleaned orders**, but account for **53.55% of positive units** and **53.45% of gross order value**. Results based on sales volume or gross value should therefore be compared with and without these orders.

The quantity histogram is limited to the 99th percentile to keep the typical distribution readable. This restriction applies only to the visualisation and does not remove any rows from the cleaned data.

### Inventory

Inventory contains 3,741 rows covering 2,500 products. A product may be stored in more than one warehouse country, so repeated product IDs are expected. The correct record identifier is the combination of product_id and warehouse_country.

After country names were standardized, the table contained no exact duplicate copies, invalid product IDs, missing warehouse countries or repeated product-country combinations. No inventory records were rejected.

Warehouse-country variants were mapped to Czech Republic, Germany and Poland. The combined key was checked again after this step because labels such as GER, Deutschland and Germany could initially appear to represent different locations. The standardization did not create any key conflicts.

Stock quantities contained no invalid, fractional or negative values. The table included 255 records with zero stock. These rows were retained because zero indicates that a product is unavailable in a specific warehouse country. Removing them would hide stock shortages.

The stock-update review identified 6 missing dates, 7 values containing only a year and month, and 13 invalid dates. Another 224 ambiguous values were interpreted as DD-MM-YYYY using clear day-first examples from the same column. Rows with incomplete or invalid update dates remain in **inventory_clean.csv**, but they are excluded from the narrower inventory-review subset.

The age of each stock update was measured against **31 December 2024**, the latest valid date found in the supplied sales and inventory datasets. This date was used only as a technical reference point. It is not a confirmed extraction date or inventory snapshot date.

Among the 3,715 records with valid update dates, 3,156 were more than 90 days old and 559 were within the 90-day threshold. No update occurred after the reference date. The table can therefore describe recorded stock quantities, but it should not be presented as a confirmed current inventory position.

## Cross-Dataset Checks and Analytical Subsets

Every product ID recorded in Sales Orders and Inventory was found in the Products table. This confirmed that product information, including categories, subcategories and base prices, could be added without creating unmatched product records.

The relationship between Sales Orders and Products was validated as many-to-one. Multiple orders may refer to the same product, while each product ID appears only once in Products. This prevents sales values from being multiplied during the merge.

Inventory remained at product and warehouse-country level because one product may be stored in several locations. It was therefore kept separate from the order-level sales table.

After validating these relationships, I created two subsets for the main sales analysis and inventory review. Each subset contains only records that meet the requirements of its intended use.

| Subset                             |    Rows | Inclusion rules                                                                                       | Intended use                                                              |
| ---------------------------------- | ------: | ----------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| **sales_orders_core_analysis.csv** | 254,804 | Valid order date, recognized country and status, positive quantity and unit price, matched product ID | Order counts and gross-value analysis by date, market, status and product |
| **inventory_stock_review.csv**     |   3,715 | Recognized warehouse country, non-negative stock, valid update date, matched product ID               | Warehouse availability and stock-freshness review                         |

Missing discounts remain in the core sales subset because gross value does not require a confirmed discount. Quantity 608 also remains visible because it requires sensitivity testing rather than structural rejection. The stock-review subset retains stale records with a flag so that freshness can be evaluated without losing recorded quantity information.

### Orders compared with product launch dates

Orders with valid order dates were compared with products that had valid launch dates. The result contained 125,257 orders placed on or after the recorded launch date and **121,535 orders** placed before it. A further 13,208 orders could not be assessed because at least one required date was unavailable.

Orders before the recorded launch date represent **49.2% of assessable orders**. This is too large to treat as a small number of isolated errors. The available source documentation does not confirm whether launch_date represents the first sale, catalogue introduction, market-specific launch or another event.

I did not modify either field. Until the business definition is confirmed, launch dates should not be used for product-age, pre-launch-sales or lifecycle conclusions.

### Transaction prices compared with product base prices

For records with positive transaction and base prices, the notebook calculates the percentage difference as:

(unit_price - base_price) / base_price × 100

The comparison was reviewed under three possible absolute-difference thresholds.

| Absolute difference from base price | Orders flagged | Share of assessable orders |
| ----------------------------------: | -------------: | -------------------------: |
|                       More than 20% |        124,903 |                     48.34% |
|                       More than 50% |         15,983 |                      6.19% |
|                      More than 100% |              8 |                      0.00% |

No threshold was used to reject an order. The source does not confirm whether differences reflect promotions, historical price changes, market-specific pricing or another definition of base_price. The comparison remains a review measure rather than a cleaning rule.

The displayed chart is restricted to the 1st–99th percentile for readability. No transaction is removed from the exported data because of this visual limit.

<!-- Add the transaction-price versus base-price visualisation here. -->

## Validation and Output Files

### Row reconciliation

The final reconciliation uses _source_row to confirm that every source record appears exactly once in either a cleaned or rejected table. A record may not appear in both groups, and no unexpected output row may be introduced.

| Dataset      | Source rows | Cleaned rows | Rejected rows | Overlap | Unaccounted | Unexpected |
| ------------ | ----------: | -----------: | ------------: | ------: | ----------: | ---------: |
| Products     |       2,500 |        2,500 |             0 |       0 |           0 |          0 |
| Sales Orders |     260,780 |      260,000 |           780 |       0 |           0 |          0 |
| Inventory    |       3,741 |        3,741 |             0 |       0 |           0 |          0 |

The final controls also confirmed unique product_id values in Products, unique order_id values in Sales Orders and unique product-country keys in Inventory. Both analytical subsets contain only rows from their parent cleaned tables, and neither subset contains an unmatched product reference.

### Export read-back validation

The notebook exports ten CSV files and then reads each file back from disk. Row counts and column order are compared with the in-memory tables. For row-level outputs, the full _source_row sequence is also checked.

**All ten exported files passed the applicable read-back controls.** This confirmed that the saved outputs retained the expected rows and structure after export.

## Summary.

| Finding                                        |                         Result | Decision                                                                |
| ---------------------------------------------- | -----------------------------: | ----------------------------------------------------------------------- |
| Exact duplicate Sales Orders copies            |                            780 | Reject the repeated copies and retain the first occurrence              |
| Structurally cleaned Sales Orders              |               **260,000 rows** | Preserve valid orders even when individual fields require review        |
| Core sales-analysis subset                     |               **254,804 rows** | Apply date, category, positive-value and product-reference requirements |
| Missing discounts after duplicate removal      |                         31,243 | Retain as unknown rather than replacing with 0%                         |
| Orders with quantity 608                       |                          1,563 | Retain with a review flag and test results with and without them        |
| Orders before the recorded product launch date |                        121,535 | Do not correct automatically; treat the field definition as unresolved  |
| Inventory records older than 90 days           | **3,156 of 3,715 review rows** | Retain with a freshness flag                                            |
| Exported files validated after saving          |                       10 of 10 | Confirm row counts, columns and source-row tracking after read-back     |

### Output files

**Cleaned data**

* data/processed/**products_clean.csv** — structurally valid product records with original values, cleaned fields and quality indicators.
* data/processed/**sales_orders_clean.csv** — structurally valid orders, including field-level statuses and review flags.
* data/processed/**inventory_clean.csv** — structurally valid product-country inventory records with quantity and freshness indicators.

**Analysis and review subsets**

* data/processed/**sales_orders_core_analysis.csv** — orders meeting the requirements of the defined core sales analysis.
* data/processed/**inventory_stock_review.csv** — inventory rows with usable product, warehouse, quantity and update-date fields.

**Rejected records**

* data/audit/**products_rejected.csv**
* data/audit/**sales_orders_rejected.csv**
* data/audit/**inventory_rejected.csv**

**Audit summaries**

* data/audit/**quality_issue_summary.csv** — counts of detected data-quality issues and review indicators.
* data/audit/**key_findings.csv** — selected findings with their analytical interpretation.

## Repository Structure and Reproduction

```text
.
├── Retail_Sales_Data_Cleaning.ipynb
├── README.md
└── data
    ├── raw
    │   ├── sales_orders.csv
    │   ├── products.csv
    │   └── inventory.csv
    ├── processed
    └── audit
```

The workflow requires Python, pandas, Matplotlib and Jupyter Notebook. The required packages can be installed with:

```bash
pip install pandas matplotlib jupyter
```

To reproduce the outputs:

1. Place sales_orders.csv, products.csv and inventory.csv in data/raw.
2. Open Retail_Sales_Data_Cleaning.ipynb from the project root.
3. Run all notebook cells in order.
4. Find cleaned and analytical outputs in data/processed.
5. Find rejected records and audit summaries in data/audit.

The notebook creates the processed and audit directories when they are not already present. It stops before transformation when a required source file or critical schema column is missing.

## Limitations and Next Step

Several limitations cannot be resolved from the supplied files alone. Missing discounts prevent confirmed net-order-value calculations. Missing, partial and invalid dates restrict time-based analysis. Quantity 608 requires sensitivity testing because it has a disproportionate effect on units and gross value. The meaning of launch_date is inconsistent with a large share of order dates, while the inventory reference date is not a confirmed snapshot date. Most valid inventory updates are also older than the 90-day review threshold.

These limitations remain visible in the exported tables rather than being hidden through unsupported corrections. The cleaned data and purpose-specific subsets will be used in a separate business analysis project covering sales, product performance and inventory.

## Tools

Python | pandas | Matplotlib | Jupyter Notebook | pathlib
