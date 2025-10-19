# Store Performance Dashboard

![alt text](/Images/Store%20Sales%20Dashboard.PNG)

This repository contains the Power BI project file and resources for a comprehensive Store Performance Dashboard. The dashboard analyzes sales, profit, and customer data across different regions, product categories, and time periods to provide actionable insights.

## Project Overview

The goal of this project is to create a dynamic dashboard that allows stakeholders to monitor key performance indicators (KPIs), understand sales trends, and identify top-performing products and regions. This dashboard answers questions like:
* What are our total sales and profit?
* Which product categories are most successful?
* What is the sales performance across different continents?
* How have sales and profit trended over time?
* Who are our best-selling individual products?

## Data Processing & Modeling

![alt text](/Images/data%20model19.PNG)

The pipeline for this project follows standard business intelligence practices, moving from raw data to a clean, modeled, and visualized report.

### 1. Data Transformation (ETL)

The initial data was spread across multiple tables (as seen in the data model). The **Power Query** editor in Power BI was used for the Extract, Ttransform, and Load (ETL) process:

* **Extraction:** Connected to data sources (e.g., SQL database, Excel files).
* **Transformation:**
    * **Cleaning:** Checked for and handled null values, removed duplicates, and corrected data types (e.g., ensuring `Quantity` is a whole number and `OrderDate` is a date).
    * **Feature Engineering:** Columns like `DeliveryDays` and `Average Delivery Days` were likely calculated by subtracting the `OrderDate` from the `DeliveryDate`.
    * **Filtering:** Removed irrelevant data to ensure analysis is based on valid sales.

### 2. Data Modeling (Snowflake Schema)

Based on the project's data model, a **Snowflake Schema** was implemented. This is an extension of a star schema where dimension tables are normalized into multiple related tables (e.g., `Products` is linked to `Subcategory`, which is linked to `Category`). This structure provides granular detail and reduces data redundancy.

The model consists of **two Fact Tables** (in a parent-child relationship) and **five Dimension Tables**.

* **Fact Tables:**
    * `Transactions`: The main "header" fact table. It records each transaction event, linking the customer (`CustomerID`), the store (`StoreID`), and the order (`OrderNumber`).
    * `Orders`: The "details" fact table. It contains the individual line items for each order, including `ProductID` and `Quantity`. It is linked to the `Transactions` table by `OrderNumber`.

* **Dimension Tables:**
    * `Customers`: Contains all customer-specific details like `CustomerName`, `CustomerGender`, and demographics (`CustomerCity`, `CustomerContinent`, `CustomerCountry`).
    * `Store`: Contains details for each store location, such as `StoreCountry`, `StoreState`, and `StoreSizeMeters`.
    * `Products`: The main product dimension, containing `ProductName`, `ProductBrand`, `ProductPrice`, and `ProductCost`.
    * `Subcategory`: A snowflake dimension linked to `Products`, containing `ProductSubcategory`.
    * `Category`: A snowflake dimension linked to `Subcategory`, containing `ProductCategory`.

This model allows for robust analysis, enabling visuals to be filtered by customer, store, product category, or any combination.

### 3. DAX Measures

To perform the calculations seen on the dashboard, several **DAX (Data Analysis Expressions)** measures were created. Since `Sales` and `Profit` are not raw columns in a single table, they must be calculated by iterating over the `Orders` table and using data from the related `Products` table.

Key measures include:
* `Total Sales = SUMX(Orders, Orders[Quantity] * RELATED(Products[ProductPrice]))`
* `Total Profit = SUMX(Orders, (Orders[Quantity] * RELATED(Products[ProductPrice])) - (Orders[Quantity] * RELATED(Products[ProductCost])))`
* `Average Delivery Days = AVERAGE(Orders[DeliveryDays])`
* `Average Store Size = AVERAGE(Store[Average Store Size])`

These measures correctly aggregate data across the related tables to power the visualizations.

## Dashboard Visualizations & Insights

The dashboard is built on a single page with a dark theme for high contrast and readability.

* **KPI Cards:**
    * **Total Sales (23.71M):** The primary revenue metric, calculated with the `Total Sales` DAX measure.
    * **Total Profit (13.91M):** The primary profitability metric, calculated with the `Total Profit` DAX measure.
    * **Average Store Size (1.38K):** A key metric for store operations, from the `Store` table.
    * **Average Delivery Days (5):** A measure of supply chain efficiency, from the `Orders` table.

![alt text](/Images/category.PNG)
* **Total Sales by Product Category (Bar Chart):**
    * **Insight:** Clearly shows that "Computers" is the dominant sales category (8.3M), followed by "Home Appliances" (4.6M). This visual uses `Total Sales` and the `Category[ProductCategory]` column.

![alt text](/Images/gender.PNG)
* **Total Sales By Gender (Donut Chart):**
    * **Insight:** The customer base is almost evenly split, with "Male" (51%) customers accounting for slightly more sales. This uses `Total Sales` and the `Customers[CustomerGender]` column.

![alt text](/Images/sales%20by%20continent.PNG)
* **Sales By Continent (Treemap):**
    * **Insight:** "North America" (15M) is by far the largest market. This visual uses `Total Sales` and the `Customers[CustomerContinent]` column.

![alt text](/Images/Best-selling%20products.PNG)
* **Best-Selling Products (Matrix):**
    * **Insight:** Provides a granular view of top products, broken down by continent. This uses `Products[ProductName]`, `Customers[CustomerContinent]`, and `Total Sales`.

![alt text](/Images/Total%20Salse%20vs%20total%20profit.PNG)
* **Total Sales vs Total Profit (Combo Chart):**
    * **Insight:** This trend chart shows the relationship between sales (bars) and profit (line) over time (2016-2021). This uses the `Total Sales` and `Total Profit` measures against a Date dimension (likely from the `Orders[OrderDate]` column).

## Tools Used

* **Power BI Desktop:** For data transformation (Power Query), data modeling, DAX calculations, and visualization.
* **DAX:** For creating the custom, aggregated measures.