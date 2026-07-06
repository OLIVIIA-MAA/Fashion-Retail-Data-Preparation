# Retail Sales Data Cleaning & Quality Assessment

## Project Overview

This project focuses on preparing raw retail sales, product and inventory data for further business analysis.
Before any meaningful analysis can be done, the data needs to be reliable. In this project, I reviewed three connected datasets, identified data quality issues, cleaned inconsistent values, standardized key fields and validated the final outputs.
The goal was not to draw business conclusions at this stage. The purpose of this notebook was to create clean, well-documented datasets that can be safely used in a separate business analysis project focused on revenue growth, pricing, product performance and inventory optimization.

## Why This Project Matters

Data cleaning is often the part of analysis that determines whether the final results can be trusted.

In this project, the raw data contained several common issues that appear in real analytical work:

- inconsistent date formats,
- missing values,
- duplicated records,
- inconsistent country names,
- text fields that needed standardization,
- numeric fields requiring validation,
- business records that needed to be separated from data quality errors.

Instead of cleaning everything automatically, each issue was reviewed in context. The main question behind every step was:
Will this issue affect the reliability of future analysis?

This helped separate actual data quality problems from valid business records, such as cancelled orders or zero stock quantities.

## Datasets Used

The project is based on three raw datasets:

### 1. Sales Orders

The sales orders dataset contains transaction-level information. It includes order dates, product identifiers, quantities, prices, discounts, countries and order statuses.

This dataset required the most attention because it will be used later for revenue, customer and product-level analysis.

### 2. Products

The products dataset works as a product reference table. It contains product identifiers, categories, subcategories, base prices and launch dates.

The main goal was to make sure that product data can be safely joined with sales and inventory data without creating duplicated or incomplete records.

### 3. Inventory

The inventory dataset contains stock information by product and warehouse country.

This dataset was reviewed to make sure that stock quantities, warehouse locations and product references are consistent enough for future inventory analysis.

## Cleaning Workflow

The project follows a structured data cleaning workflow:
1. Load raw datasets.
2. Review dataset structure and data types.
3. Check missing values.
4. Identify duplicate records.
5. Standardize date columns.
6. Standardize categorical fields.
7. Validate numerical columns.
8. Check relationships between datasets.
9. Create cleaned datasets.
10. Run final validation checks.
11. Export cleaned files for further analysis.

Each step was documented in the notebook to make the cleaning process transparent and easy to follow.

## Main Cleaning Steps

### Sales Orders Dataset

The sales orders dataset was prepared as the main transaction table.

The cleaning process included:

- reviewing missing values,
- removing technical duplicate rows,
- standardizing the `order_date` column,
- removing records without valid order dates,
- standardizing country names,
- validating quantity and unit price values,
- converting discount values into numeric format,
- standardizing order status labels.

Cancelled orders were not treated as data errors. They were kept in the cleaned sales dataset because they are valid historical records. A separate revenue-ready dataset was created for future analysis, where cancelled orders can be excluded from revenue calculations.
This approach keeps the original business context while still preparing a clean dataset for analysis.

### Products Dataset

The products dataset was cleaned as a product reference table.
The cleaning process included:

- checking product identifier uniqueness,
- reviewing missing values,
- checking duplicate records,
- standardizing product categories and subcategories,
- validating base prices,
- converting launch dates into datetime format,
- creating a `launch_year` column,
- checking whether product IDs from sales orders exist in the products table.

Missing launch dates were kept as missing values. A missing launch date does not make the full product record invalid, but it needs to be considered separately in analyses that depend on launch timing.

### Inventory Dataset

The inventory dataset was prepared for future stock and inventory analysis.
The cleaning process included:

- validating product identifiers,
- reviewing missing values,
- checking duplicate records,
- checking duplicated product and warehouse combinations,
- standardizing warehouse country names,
- validating stock quantity values,
- reviewing zero stock records,
- converting stock update dates into datetime format,
- checking whether inventory product IDs exist in the products table.

Zero stock values were kept in the cleaned dataset because they can represent valid out-of-stock situations. Missing or unparseable stock update dates were also retained because they do not invalidate the stock quantity itself.

## Data Quality Decisions

A key part of this project was deciding what should be removed, what should be corrected and what should remain in the dataset.

The most important decisions were:

- Duplicate rows were removed because they could lead to double-counting.
- Invalid or missing order dates were removed from the sales orders dataset because they could not be assigned to a reliable time period.
- Cancelled orders were retained in the cleaned sales dataset, but separated from the revenue-ready dataset.
- Missing launch dates were kept because products can still be valid without launch date information.
- Zero stock values were kept because they may represent valid out-of-stock cases.
- Missing stock update dates were kept as incomplete metadata rather than removed as invalid inventory records.

These decisions were made to avoid unnecessary data loss while keeping the final datasets reliable.

## Output Datasets

The cleaning process creates cleaned versions of the original datasets: 
- sales_orders_clean.csv
- sales_orders_revenue.csv
- products_clean.csv
- inventory_clean.csv

The **sales_orders_clean** file keeps all valid order statuses, including cancelled orders.
The **sales_orders_revenue** file excludes cancelled orders and is intended for future revenue analysis.

## Project Structure

```text
retail-sales-data-cleaning/
│
├── data/
│   ├── raw/
│   │   ├── sales_orders.csv
│   │   ├── products.csv
│   │   └── inventory.csv
│   │
│   └── processed/
│       ├── sales_orders_clean.csv
│       ├── sales_orders_revenue.csv
│       ├── products_clean.csv
│       └── inventory_clean.csv
│
├── notebooks/
│   └── 01_data_cleaning.ipynb
│
├── README.md
└── requirements.txt
