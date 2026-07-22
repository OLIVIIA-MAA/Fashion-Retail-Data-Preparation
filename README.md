# Retail Sales Data Cleaning & Quality Assessment

## Project Overview

This repository presents the data cleaning stage of a retail analytics project. The raw data consisted of three connected CSV files: sales orders, product information and inventory records. Together, these datasets describe what was sold, what products were available in the offer and how stock was distributed across warehouse countries.
Before starting the business analysis, the data needed to be checked and prepared. The main focus of this project was to understand the structure of each dataset, identify quality issues, clean inconsistent values and create reliable files for the next analytical stage.

This repository does not focus on charts or business recommendations yet. It shows the work that happens before analysis: reviewing raw data, making cleaning decisions and validating whether the final datasets are ready to be used.

## Data Overview

The project contains three datasets connected through product_id.

| Dataset | Final rows | Dataset grain | Main purpose |
|---|---:|---|---|
| Sales Orders | 255,486 | One row per order | Sales, revenue and order status analysis |
| Products | 2,500 | One row per product | Product category, pricing and launch analysis |
| Inventory | 3,741 | One row per product and warehouse country | Product availability and stock analysis |

Understanding the grain of each table was important before checking duplicates or joining the datasets. A repeated product_id is incorrect in the Products table because each product should appear once. The same repeated product_id can be valid in the Inventory table because one product may be stored in several warehouse countries.
