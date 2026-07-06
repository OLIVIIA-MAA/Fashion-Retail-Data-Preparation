# Retail Sales Data Cleaning & Quality Assessment

## Project Overview

This project shows the full data cleaning process for a retail dataset containing sales orders, product information and inventory records. The main goal was to turn raw, inconsistent CSV files into clean and validated datasets ready for further business analysis.

I focused on the preparation stage of an analytics project: checking data quality, identifying issues, cleaning inconsistent values, validating key fields and creating final output files. The project does not focus on business conclusions yet. Its purpose is to show how the data was prepared before moving into revenue, product and inventory analysis.

This repository is the first part of a larger retail analytics project. The cleaned files created here will be used in a separate analysis project focused on revenue growth, pricing, product performance and inventory efficiency.

## What This Project Shows

This project demonstrates that I can work with raw data before the analysis stage. I reviewed three connected datasets, checked their quality and prepared them for reliable use in future analysis.

The main work included checking missing values, detecting duplicates, standardizing date formats, cleaning inconsistent country names, validating prices and quantities, standardizing categorical fields and checking relationships between datasets.

I also documented cleaning decisions clearly, especially in cases where the data required business context rather than automatic removal. For example, cancelled orders and zero stock values were not treated as errors because they can be valid business records.

## Datasets Used

The project uses three raw datasets.

The sales orders dataset contains transaction-level data, including order dates, product IDs, quantities, unit prices, discounts, countries and order statuses. This was the most important dataset to clean because it will later be used for revenue and customer analysis.

The products dataset works as a product reference table. It contains product IDs, categories, subcategories, base prices and launch dates. The goal was to make sure that product information could be safely joined with sales and inventory data.

The inventory dataset contains stock information by product and warehouse country. It was cleaned to support later inventory analysis and to make sure stock records could be connected with product data.

## Cleaning Process

The cleaning process was organized by dataset. Each dataset was reviewed separately because each one had a different role in the future analysis.

For the sales orders dataset, I checked missing values, duplicate rows, order dates, country names, quantities, unit prices, discounts and order statuses. Invalid or missing order dates were removed because they could not be used in time-based analysis. Cancelled orders were kept in the cleaned sales dataset, but I also created a separate revenue-ready version where cancelled orders are excluded.

For the products dataset, I checked product ID uniqueness, missing values, duplicate records, category consistency, base prices and launch dates. Mixed launch date formats were converted to datetime, and a launch year column was created for later analysis.

For the inventory dataset, I checked product IDs, missing values, duplicate records, product and warehouse combinations, warehouse country names, stock quantities and stock update dates. Zero stock values were kept because they can represent real out-of-stock situations.

## Main Data Quality Decisions

Some records were removed because they could affect the reliability of future analysis. Duplicate rows were removed to avoid double-counting, and sales records without valid order dates were removed because they could not be assigned to a reliable month or year.

Some records were kept because they represented valid business situations. Cancelled orders were kept in the cleaned sales dataset, zero stock values were kept in the inventory dataset, and missing launch dates were kept in the products dataset. These values were documented instead of being removed automatically.

This approach helped avoid unnecessary data loss while still preparing datasets that are reliable enough for further analysis.

## Output Files

The project creates four cleaned output files:

- `sales_orders_clean.csv` contains cleaned sales orders with all valid order statuses retained.
- `sales_orders_revenue.csv` contains sales records prepared for revenue analysis, with cancelled orders excluded.
- `products_clean.csv` contains the cleaned product reference table.
- `inventory_clean.csv` contains the cleaned inventory dataset.

The cleaned files are saved in the `data/processed/` folder and are ready to be used in the next stage of the project.

## Skills Demonstrated

This project demonstrates practical data cleaning and data preparation skills:

- data quality assessment,
- missing value analysis,
- duplicate detection,
- date parsing and standardization,
- categorical data cleaning,
- numerical value validation,
- working with related datasets,
- checking relationships between tables,
- preparing analysis-ready files,
- documenting cleaning decisions clearly.

## Tools Used

- Python
- Pandas
- NumPy
- Google Colab / Jupyter Notebook

## Repository Structure

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
