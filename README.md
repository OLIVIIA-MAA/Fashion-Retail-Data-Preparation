# Retail Sales Data Cleaning & Quality Assessment

## Project Overview

This repository presents the data cleaning stage of a retail analytics project. The raw data consisted of three connected CSV files: sales orders, product information and inventory records. Together, these datasets describe what was sold, what products were available in the offer and how stock was distributed across warehouse countries.
Before starting the business analysis, the data needed to be checked and prepared. The main focus of this project was to understand the structure of each dataset, identify quality issues, clean inconsistent values and create reliable files for the next analytical stage.

This repository does not focus on charts or business recommendations yet. It shows the work that happens before analysis: reviewing raw data, making cleaning decisions and validating whether the final datasets are ready to be used.

## About the Data

The project uses three datasets that play different roles in the analysis.

The sales orders dataset is the transaction table. It contains order-level information such as order dates, product IDs, quantities, unit prices, discounts, countries and order statuses. Because this dataset will later be used to calculate revenue and analyze customer behavior, it required the most detailed quality checks.
The products dataset works as a reference table for product information. It contains product IDs, categories, subcategories, base prices and launch dates. This dataset is important because it adds product context to sales and inventory records.

The inventory dataset describes stock levels by product and warehouse country. It contains product IDs, warehouse locations, stock quantities and the date of the last stock update. This dataset will later support inventory and stock distribution analysis.

## Main Data Quality Issues

The raw datasets contained several issues that needed to be handled before analysis.
In the sales orders data, the main problems were mixed date formats, missing or invalid order dates, inconsistent country names, discount values stored as text and order statuses written in different formats. The dataset also contained cancelled orders, which required a business decision rather than simple removal.

In the products data, the structure was more stable, but the launch date column needed attention. Dates appeared in different formats, and some products did not have launch date information. Product categories and base prices were also reviewed to make sure the product table could be used safely as a reference table.
In the inventory data, the biggest issue was inconsistent warehouse country naming. The same countries appeared under different names and abbreviations, such as local-language names, shortened codes and different capitalization. The stock update date column also contained mixed date formats and missing values. Stock quantities were reviewed separately because zero stock can be a valid business situation, not a data error.


## Cleaning Approach

The cleaning process was organized around the role of each dataset.

For sales orders, I prepared the transaction data for future revenue and customer analysis. I removed duplicate rows, standardized order dates, removed records without valid order dates, cleaned country names, validated quantities and unit prices, converted discount values to numeric format and standardized order statuses.
Cancelled orders were kept in the cleaned sales dataset because they are part of the order history. At the same time, I created a separate revenue-ready version of the sales data where cancelled orders are excluded. This makes the data more flexible for later analysis and avoids mixing historical order status with revenue calculations.

For products, I focused on making the table reliable as a product reference source. I checked whether product IDs were unique, reviewed missing values, standardized category fields, validated base prices and converted launch dates to datetime format. I also created a launch year column so that product launch timing can be used more easily in the next stage of analysis.

For inventory, I cleaned warehouse country names, checked duplicate records, validated stock quantities and converted stock update dates to datetime format. I also checked whether product IDs from the inventory table existed in the cleaned products table, because inventory records need product context to be useful in later analysis.


## Important Cleaning Decisions

Not every unusual value was treated as an error. Some records needed to be interpreted in context.

Cancelled orders were not removed from the cleaned sales dataset because they are valid business records. They were only excluded from the separate revenue-ready file.

Zero stock values were kept in the inventory dataset because they can represent products that are currently out of stock.

Missing launch dates were kept in the products dataset because a product can still be useful for category, pricing and inventory analysis even if its launch date is unavailable.

Missing or unparseable stock update dates were also kept because they affect information about stock freshness, but they do not invalidate the stock quantity itself.

This approach helped keep useful business information in the data instead of removing records too aggressively.


## Output Files

The project produces four cleaned files:

- sales_orders_clean.csv — cleaned transaction data with all valid order statuses retained,
- sales_orders_revenue.csv — sales data prepared for revenue analysis, with cancelled orders excluded,
- products_clean.csv — cleaned product reference table,
- inventory_clean.csv — cleaned inventory dataset.

These files are saved in the data/processed/ folder and are intended to be used in the next part of the project.

## Skills and Techniques Used

This project covers practical data cleaning work in Python, including missing value review, duplicate detection, date parsing, categorical standardization, numerical validation and checks between related tables.
The project also shows how cleaning decisions were made based on the future use of the data. Instead of applying the same rule to every issue, each problem was reviewed in context: whether it affected revenue analysis, product analysis, inventory reporting or dataset joins.

## Tools Used

- Python
- Pandas
- NumPy
- Jupyter Notebook

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
