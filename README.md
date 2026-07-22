# Retail Sales Data Cleaning and Quality Assessment

## Project Overview

Reliable business analysis begins before the first metric is calculated. Inconsistent categories, incorrectly interpreted identifiers or incomplete transaction data can affect revenue, market comparisons and product performance even when the analytical code itself runs correctly.

This project prepares three connected retail datasets for further business analysis:

| Dataset      | Raw records | Cleaned records | Main analytical purpose                              |
| ------------ | ----------: | --------------: | ---------------------------------------------------- |
| Sales Orders |     260,780 |         255,486 | Revenue, sales volume, markets, products and trends  |
| Products     |       2,500 |           2,500 | Product categories, pricing and launch analysis      |
| Inventory    |       3,741 |           3,741 | Product availability, shortages and stock allocation |

The cleaning process was based on how each table would later be used. Missing values, repeated identifiers and unusual records were not removed automatically. I first determined whether they represented incorrect data, valid business information or a limitation that should remain visible in the final datasets.

As a result, records were removed only when they could not be interpreted reliably for the planned analysis. Valid information such as cancelled orders, products without launch dates and zero-stock records was preserved.

## Tools and Workflow

The cleaning process was completed in Python using pandas and NumPy. Pathlib was used to manage the project folders and file paths.

The notebook follows a reproducible sequence:

* load the three raw CSV files
* create working copies to preserve the original data
* review structure, data types and missing values
* validate identifiers and table-level granularity
* standardise dates and categorical values
* review numerical fields used in calculations
* verify relationships between the datasets
* run final validation checks
* export the cleaned datasets for further analysis

## Sales Orders

Sales Orders is the central dataset in the project and the basis for revenue, product, market and time-based analysis. The raw table contained 260,780 records. Before calculating any metrics, I needed to determine which rows represented usable sales transactions and which could distort transaction counts or financial results.

### Cleaning impact

| Cleaning stage             | Rows before | Records removed | Rows after | Decision                                                      |
| -------------------------- | ----------: | --------------: | ---------: | ------------------------------------------------------------- |
| Full-row duplicate removal |     260,780 |             780 |    260,000 | Remove identical technical duplicates                         |
| Order date validation      |     260,000 |           1,892 |    258,108 | Remove orders without a reliable reporting date               |
| Quantity validation        |     258,108 |           1,027 |    257,081 | Exclude non-positive quantities from standard sales analysis  |
| Unit price validation      |     257,081 |           1,595 |    255,486 | Exclude non-positive prices from revenue and pricing analysis |

In total, 5,294 records were removed, representing 2.03% of the raw table. The final 255,486 records are suitable for standard sales analysis under the assumptions described below.

### Confirming what one row represents

A repeated order identifier could indicate a duplicated transaction, but it could also mean that one order was divided across several product-level rows. I therefore reviewed both complete row duplication and identifier uniqueness before deciding whether any aggregation was required.

The raw table contained 780 rows that were identical across all nine columns. Because there were no differences that would suggest separate business events, I classified them as technical duplicates and removed them.

After this step, the table contained 260,000 rows and 260,000 unique order identifiers. No order identifiers were missing or repeated. This confirmed that each row represents one complete order and that the dataset does not require additional aggregation before analysis.

### Establishing reliable reporting dates

The initial review identified 629 empty order dates. However, the column was stored as text and contained different formats and separators, so the original missing-value count did not show whether all remaining values could be interpreted correctly.

I standardised the separators and converted the column using mixed-format parsing with day-first interpretation. Values that could not be interpreted reliably were converted to missing dates rather than being silently assigned an incorrect date.

After conversion, 1,892 records had no valid order date. This included the original missing values and additional text values that could not be parsed reliably.

I removed these records because they could not be assigned to the correct month or year. Keeping them would allow them to contribute to overall sales totals while excluding them from the correct reporting period, weakening analyses of growth, seasonality and year-over-year performance.

All remaining order dates fall between 1 January 2015 and 31 December 2024.

### Reviewing quantities and prices

Quantity and unit price directly affect sales volume and order value. Both fields therefore required validation before they could be used in calculations.

The quantity review identified 1,027 records with zero or negative values. The dataset did not include a transaction type that would allow these records to be confirmed as returns, corrections or another separate business process. I therefore excluded them from a dataset intended for standard sales analysis rather than assigning them an unsupported interpretation.

The same approach was applied to 1,595 records with non-positive unit prices. Without additional information, these values could represent errors, free products, replacements or accounting adjustments. They were unsuitable for standard revenue and pricing calculations and were removed.

After cleaning, quantities range from 1 to 608 and unit prices from 24.08 to 769.72. Neither field contains missing or non-positive values.

### Preserving incomplete discount information

The discount column was imported as text and included percentage symbols and unnecessary spaces. I removed the formatting and converted the field to a numeric type.

Before conversion, 30,688 values were missing. The same number remained missing afterwards, confirming that the transformation did not cause any valid entries to be lost.

The cleaned dataset contains 224,798 recorded discounts ranging from 0% to 60.58%. No recorded values fall outside the valid range of 0% to 100%.

I did not replace missing discounts with zero. A value of zero confirms that no discount was applied, while a missing value means that the information is unavailable. Treating both cases as equivalent would incorrectly classify 30,688 orders as full-price transactions and could bias comparisons between discounted and non-discounted sales.

The missing values were therefore retained as unknown. Analyses that do not depend on discount information can use the complete cleaned table, while discount-related results should be calculated only from confirmed values.

### Standardising markets and order statuses

Country names were recorded using abbreviations, local-language names and inconsistent capitalisation. I mapped these variants into ten consistent markets so that one country would not be divided across several reporting groups.

Order statuses were consolidated into three values:

* COMPLETED
* SHIPPED
* CANCELLED

The cleaned dataset contains 15,457 cancelled orders. I retained them because they represent real business events and may support analysis of cancellation patterns across products, markets or reporting periods.

For this project, COMPLETED and SHIPPED orders are included in revenue analysis, while CANCELLED orders are excluded. Applying this rule during analysis preserves the complete order history without allowing cancelled transactions to overstate revenue.

### Final Sales Orders result

The final sales_orders_clean dataset contains 255,486 orders and nine columns. It has:

* no full-row duplicates
* complete and unique order identifiers
* no missing or invalid order dates
* no non-positive quantities or unit prices
* no out-of-range values among recorded discounts
* no unexpected country or order status values

Missing discounts remain as an intentional limitation rather than being replaced with unsupported values.

## Products

Products is the reference table used to add category, subcategory, pricing and launch information to sales and inventory records. Because one product record may be matched with many transactions, the most important initial check was whether each product identifier referred to one product only.

### Cleaning impact

| Validation           | Before cleaning | Cleaning decision                        |             After cleaning |
| -------------------- | --------------: | ---------------------------------------- | -------------------------: |
| Product records      |           2,500 | Retain all valid products                |                      2,500 |
| Missing launch dates |              91 | Standardise and parse mixed date formats | 101 missing or unparseable |
| Valid launch dates   |           2,409 | Retain only confirmed dates              |                      2,399 |
| Dataset columns      |               5 | Add launch_year from confirmed dates     |                          6 |

No records were removed from the table.

### Validating the product reference

The table contained 2,500 rows and 2,500 unique product identifiers. No identifiers were missing, no product IDs were duplicated and no full-row duplicates were found.

This confirmed that each row represents one product. A duplicated product identifier could match one sales transaction to several reference records and inflate results after a join, so verifying uniqueness was necessary before combining the datasets.

I also compared product identifiers across the related tables. Every product referenced in Sales Orders was available in Products, so sales records can receive product attributes without losing transactions or creating unmatched product references.

### Identifying unreliable launch dates

Launch dates were stored as text and used different formats and separators. The initial review showed 91 missing values, but this did not confirm that every remaining text value represented a valid date.

After standardising and converting the column, the number of missing or unparseable dates increased from 91 to 101. The conversion therefore identified 10 additional values that could not be interpreted reliably.

I retained all affected products because a missing launch date does not invalidate their product identifiers, categories, subcategories or prices. Removing them would reduce product coverage and could cause valid sales or inventory records to lose their product attributes after a join.

I also avoided estimating the missing dates because the source data did not provide enough information to assign them reliably. Instead, launch_year was created only from confirmed launch dates.

The cleaned table contains 2,399 valid launch dates covering the period from 2 January 2015 to 30 December 2024. The remaining 101 products retain missing launch_date and launch_year values.

### Standardising product groups

Category and subcategory values were complete but required consistent formatting. I removed unnecessary spaces and applied consistent capitalisation.

The final table contains five categories and 22 subcategories, with no missing or unexpected labels. This prevents the same product group from being divided during revenue, pricing or sales volume aggregation.

### Validating base prices

All 2,500 products contained a base price. The values ranged from 31.23 to 409.97, with no missing, zero or negative prices.

No price corrections or record removals were required.

### Final Products result

The final products_clean dataset contains all 2,500 products and six columns. It provides a unique and complete product reference for identifiers, categories, subcategories and prices.

The only remaining limitation is incomplete launch information for 101 products. These records remain fully usable for analyses that do not depend on launch timing.

## Inventory

Inventory contains 3,741 stock records for 2,500 products. Unlike Products, this table is not expected to contain one row per product because the same item may be stored in more than one warehouse country.

The most important decision was therefore defining the correct inventory key before treating repeated product identifiers as duplicates.

### Cleaning impact

| Validation                              | Before cleaning | Cleaning decision                             |            After cleaning |
| --------------------------------------- | --------------: | --------------------------------------------- | ------------------------: |
| Inventory records                       |           3,741 | Retain all valid stock records                |                     3,741 |
| Warehouse country labels                |              16 | Standardise abbreviations and naming variants |                         3 |
| Duplicated product-country combinations |               0 | Recheck after country standardisation         |                         0 |
| Zero-stock records                      |             255 | Retain as valid out-of-stock information      |                       255 |
| Missing update dates                    |               6 | Standardise and parse mixed date formats      | 19 missing or unparseable |
| Invalid stock quantities                |               0 | No correction required                        |                         0 |

No inventory records were removed.

### Defining the inventory key

The table contained 3,741 records but only 2,500 unique product identifiers. Removing every repeated product ID would delete valid information about products stored in different locations.

I therefore defined one inventory record using the combination of product_id and warehouse_country. This composite key represents one product in one warehouse country and provides the correct level for duplicate detection.

No duplicated product-country combinations were found. Repeated product identifiers therefore reflect the structure of the inventory rather than duplicated stock records.

### Standardising warehouse locations

The warehouse_country field initially contained 16 labels, including abbreviations, local-language names and inconsistent capitalisation. I mapped these variants into three consistent values:

* Czech Republic
* Germany
* Poland

I repeated the composite-key check after standardisation. This was necessary because values such as GER, Deutschland and Germany could initially make two records appear to belong to different locations, even though they referred to the same country.

No duplicated product-country combinations appeared after standardisation. The transformation improved warehouse-level grouping without creating conflicting inventory records.

### Preserving zero-stock information

Stock quantities ranged from 0 to 567 and contained no missing or negative values. No quantity corrections or removals were required.

The table included 255 zero-stock records, representing 6.82% of the inventory. I retained them because zero is a valid business value that identifies a product as unavailable in a particular warehouse country.

Removing these records would hide shortages and make product availability appear better than it was.

### Identifying unreliable stock update dates

The last_stock_update field was stored as text and used several formats and separators. The initial review identified 6 missing values.

I standardised the separators and converted the field using mixed-format parsing with day-first interpretation. After conversion, 19 values were missing or invalid. This means that the transformation identified 13 additional text values that could not be interpreted reliably.

These values were converted to missing dates rather than being assigned an uncertain timestamp.

I retained the affected stock records because the missing update date does not invalidate the recorded quantity. They remain suitable for general availability and shortage analysis, although they should be excluded or reviewed separately when conclusions depend on stock freshness.

The 3,722 confirmed update dates cover the period from 12 May 2023 to 31 December 2024.

### Final Inventory result

The final inventory_clean dataset contains all 3,741 records and four columns. It has:

* no full-row duplicates
* no missing product identifiers
* no duplicated product-country combinations
* no missing or negative stock quantities
* three consistent warehouse countries
* 255 retained zero-stock records
* 19 missing or unparseable stock update dates

Every product referenced in Inventory is available in the Products table.

## Cross-Dataset Validation

The relationships between the datasets were checked before export.

All product identifiers used in Sales Orders and Inventory were found in Products. This means that the cleaned tables can be joined without losing product attributes or creating unmatched product references.

The validated table structure is:

* one Sales Orders row represents one order
* one Products row represents one product
* one Inventory row represents one product in one warehouse country

These definitions establish the correct level of detail for joins and prevent duplicated results during aggregation.

## Remaining Analytical Considerations

The cleaning process does not remove every limitation from the source data. The following rules should remain visible in the next stage of analysis:

* CANCELLED orders must be excluded from revenue calculations
* discount analysis should use only confirmed discount values
* launch-based analysis has incomplete information for 101 products
* freshness-based inventory analysis has incomplete dates for 19 records
* removed non-positive quantities and prices cannot be classified as returns or corrections because the source data did not contain the necessary transaction context

Keeping these limitations documented prevents incomplete information from being treated as confirmed business data.

## Output Files

The cleaning workflow produces three files in the processed data folder:

* sales_orders_clean.csv
* products_clean.csv
* inventory_clean.csv

Cancelled orders remain in sales_orders_clean. They are filtered out only when revenue is calculated, so the project retains one complete cleaned order history rather than maintaining a separate revenue dataset.

## Final Result

The project transforms three connected raw datasets into a consistent analytical foundation without removing valid business information unnecessarily.

The main outcome is not only cleaner formatting. The final datasets have clearly defined levels of detail, verified relationships and documented rules for records that require special treatment. This allows later revenue, product and inventory analysis to be performed without hidden duplication, inconsistent grouping or unsupported assumptions.

