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

Products is the reference table used to add categories, subcategories, base prices and launch information to Sales Orders and Inventory. Its key must be unique because one duplicated product record could multiply order values after a join.

The table contained no exact duplicate copies. All 2,500 product IDs were present, numeric in format and unique after normalization. No product row was rejected, and identifiers remained stored as text because they are labels used for matching rather than values used in arithmetic.

All category and subcategory values were present and recognized. The standardization review was empty, which means that the current file did not contain a category label requiring replacement. All base-price values could also be converted to numeric form. No missing, invalid, zero or negative base price was found, so the notebook did not correct or reject any product based on price.

Launch dates required a different decision because an unreliable date does not invalidate the product itself. The source profile contained 91 empty launch dates. After parsing, the cleaned table contained 91 missing dates, 3 partial monthly values and 10 invalid dates. Another 148 day-month values were textually ambiguous but could be parsed as DD-MM-YYYY using evidence from the same column. No ambiguous date remained unresolved after applying that rule.

I retained products with missing, partial or invalid launch dates because their IDs, categories, subcategories and prices remain useful. Estimating a missing date would create unsupported information. The cleaned table therefore keeps the original value, parsed date, detected format and date status. Analyses based on launch timing should use only products with launch_date_status = valid.

### Sales Orders: duplicate and key validation

Sales Orders contained 260,780 source rows and 780 records that were exact copies across all nine source columns. The repeated copies were moved to **sales_orders_rejected.csv**, while the first occurrence remained in the cleaned table.

After duplicate removal, **260,000 rows** remained. The notebook found no missing or invalid order IDs, customer IDs or product IDs. It also found no conflicting order-ID groups and no remaining duplicate order IDs. This confirmed that one row represents one complete order rather than one line of a multi-product order, so no additional aggregation was required before analysis.

The structurally cleaned table retains orders with reliable identities even when another field cannot support every analytical use. This prevents a missing date or unresolved numeric value from removing otherwise valid order information.

### Sales Orders: dates and categories

The cleaned Sales Orders table contains 218,047 valid dates in YYYY-MM-DD format and 39,367 valid dates in DD-MM-YYYY format. It also contains 1,263 invalid dates, 694 partial monthly values and 629 missing dates.

The column included 23,798 values that provided unambiguous evidence for the day-first convention. Based on this pattern, 15,569 textually ambiguous dates were parsed as DD-MM-YYYY and retained with an ambiguity flag.

Missing, partial and invalid dates remain in **sales_orders_clean.csv** because the order itself is still identifiable. They are excluded from **sales_orders_core_analysis.csv**, where a complete date is required for monthly, annual and seasonal reporting.

Country and order-status values were standardized through the explicit dictionaries. The mappings changed 188,960 country entries and 220,033 status entries. After validation, no missing or unrecognized country or status remained.

Cancelled orders were preserved because cancellation is a business event rather than a technical error. They may be useful for cancellation analysis, but they should not be included automatically in revenue calculations.

### Sales Orders: numeric values and discounts

Quantities were converted to nullable integers after checking their format, fractional values and sign. No invalid, fractional or negative quantity was found, but 1,034 orders contained quantity equal to zero.

Unit prices were converted to nullable floating-point values. No invalid or negative price was found, but 1,608 orders contained a zero unit price. The source files do not explain whether these zeros represent corrections, free items, replacements or another business process, so I did not assign one interpretation.

Zero-quantity and zero-price rows remain in the broad cleaned table with flags. They are excluded from the core sales subset because they cannot support standard quantity or value calculations. gross_order_value is calculated only when both quantity and unit price are positive.

The source profile contained 31,322 empty discount cells. After removal of exact duplicate copies, **31,243 missing discounts** remained. No recorded discount had an invalid format or fell outside the allowed range from 0 to 100.

I kept missing discounts as null. A recorded zero confirms that no discount was applied, while a missing value means that the discount is unknown. Replacing all missing values with zero would classify unconfirmed orders as full-price transactions. Gross value can still be calculated for these records, but net_order_value remains missing unless the discount is confirmed.

### Sales Orders: quantity 608

The positive quantity distribution has a median of 3, a 95th percentile of 6 and a 99th percentile of 8. The maximum is 608.

Quantity 608 appears in **1,563 orders**. In comparison, quantities 9, 10 and 11 appear in 408, 100 and 25 orders. The repeated value is therefore not an isolated extreme observation. However, the available fields do not show whether it represents valid bulk orders, a source-system code or another issue.

I retained the records and created quantity_608_review_flag instead of labelling them as errors. Although they represent only **0.60% of structurally cleaned orders**, they account for **53.55% of recorded positive units** and **53.45% of gross order value**. Any conclusion based on volume or gross value should therefore be compared with and without these rows.

The notebook's quantity histogram is limited to values up to the 99th percentile so that the typical distribution remains readable. This restriction affects only the chart and does not filter the cleaned data.

### Inventory

Inventory contains 3,741 rows for 2,500 products. Repeated product IDs are expected because the same product can be stored in more than one warehouse country. Product ID alone is therefore not the correct duplicate key.

One inventory record is defined by the combination of product_id and standardized warehouse_country. The table contained no exact duplicate copies, invalid product IDs, missing warehouse countries or duplicate product-country combinations after standardization. No inventory row was rejected.

Warehouse labels were mapped to Czech Republic, Germany and Poland. The composite key was checked again after mapping because values such as GER, Deutschland and Germany could initially make two records appear to belong to separate locations. Standardization did not create any conflicting keys.

Stock quantities contained no invalid formats, fractional values or negative values. The table did contain 255 zero-stock records. I retained them because zero is a valid business value that identifies a product as unavailable in a warehouse country. Removing these rows would hide shortages.

The stock-update review found 6 missing dates, 7 partial monthly values and 13 invalid dates. Another 224 day-month values were textually ambiguous but could be parsed as DD-MM-YYYY using evidence from the same column. Rows with incomplete or invalid update dates remain in **inventory_clean.csv**, but they are excluded from the narrower stock-review subset.

Inventory age is calculated against 31 December 2024, the latest valid date found in the supplied sales and inventory files. This is a project reference date, not a confirmed extraction or inventory snapshot date.

A stock update older than 90 days receives stale_stock_update_review_flag. Among the 3,715 records suitable for stock review, 3,156 were older than 90 days and 559 were within the threshold. No update occurred after the reference date. The table can describe recorded quantities, but it should not be presented as a confirmed current stock position without a verified snapshot date or update schedule.

## Cross-Dataset Checks and Analytical Subsets

All product IDs recorded in Sales Orders and Inventory were successfully matched with the Products table. This confirmed that product information, including categories, subcategories and base prices, could be added without losing records due to missing product references.

The relationship between Sales Orders and Products was verified as many-to-one. Multiple orders may relate to the same product, while each product ID appears only once in the Products table. This prevents sales values from being duplicated during the merge. Inventory was kept at product and warehouse-country level because the same product may be stored in several locations.

After these relationship checks, I created two separate subsets for the main sales analysis and inventory review. Each subset contains only the records that meet the requirements of its intended analysis.

Two purpose-specific subsets were created after structural cleaning.

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
