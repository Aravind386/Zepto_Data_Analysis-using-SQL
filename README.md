
# 🛒 Zepto Inventory and Product Analysis Using SQL

## Overview

This project focuses on analyzing Zepto product and inventory data using PostgreSQL. The objective is to explore product pricing, discounts, stock availability, inventory weight, and estimated revenue to derive meaningful business insights.

The project includes data exploration, data quality checks, data cleaning, and business analysis using SQL queries. It demonstrates how SQL can help evaluate product pricing strategies, identify stock availability issues, compare product value, and analyze inventory across categories.

## Objectives

- Explore the Zepto product dataset and understand its structure.
- Identify missing values and duplicate product names.
- Analyze product availability and stock status.
- Clean invalid price records and convert prices from paise to rupees.
- Identify products offering the highest discounts.
- Find high-priced products that are out of stock.
- Estimate revenue by product category.
- Analyze categories with the highest average discounts.
- Calculate price per gram to compare product value.
- Classify products based on weight.
- Calculate total inventory weight by category.

## Dataset

The project uses Zepto product and inventory data.

- **Database:** PostgreSQL
- **Table Name:** `zepto`
- **Dataset Source:** Add the original dataset link here if it was downloaded from Kaggle or another source.

### Dataset Columns

| Column | Description |
|---|---|
| `sku_id` | Unique identifier for each SKU |
| `category` | Product category |
| `name` | Product name |
| `mrp` | Maximum retail price |
| `discountPercent` | Discount percentage offered |
| `availableQuantity` | Available inventory quantity |
| `discountedSellingPrice` | Selling price after discount |
| `weightInGms` | Product weight in grams |
| `outOfStock` | Indicates whether the product is out of stock |
| `quantity` | Quantity field recorded in the dataset |

## Database Schema

The following SQL creates the table used for the analysis.

```sql
DROP TABLE IF EXISTS zepto;

CREATE TABLE zepto (
    sku_id SERIAL PRIMARY KEY,
    category VARCHAR(120),
    name VARCHAR(150) NOT NULL,
    mrp NUMERIC(8,2),
    discountPercent NUMERIC(5,2),
    availableQuantity INTEGER,
    discountedSellingPrice NUMERIC(8,2),
    weightInGms INTEGER,
    outOfStock BOOLEAN,
    quantity INTEGER
);
```

## Data Exploration

### 1. Count Total Records

**Objective:** Determine the total number of records in the dataset.

```sql
SELECT COUNT(*) AS total_records
FROM zepto;
```

### 2. View Sample Data

**Objective:** Inspect a sample of product records to understand the dataset.

```sql
SELECT *
FROM zepto
LIMIT 10;
```

### 3. Identify Missing Values

**Objective:** Find records with missing values in important product fields.

```sql
SELECT *
FROM zepto
WHERE name IS NULL
   OR category IS NULL
   OR mrp IS NULL
   OR discountPercent IS NULL
   OR discountedSellingPrice IS NULL
   OR weightInGms IS NULL
   OR availableQuantity IS NULL
   OR outOfStock IS NULL
   OR quantity IS NULL;
```

### 4. Explore Product Categories

**Objective:** Identify the distinct product categories in the dataset.

```sql
SELECT DISTINCT category
FROM zepto
ORDER BY category;
```

### 5. Analyze In-Stock vs. Out-of-Stock Products

**Objective:** Understand the distribution of products by stock status.

```sql
SELECT
    outOfStock,
    COUNT(sku_id) AS total_products
FROM zepto
GROUP BY outOfStock;
```

### 6. Find Product Names with Multiple SKUs

**Objective:** Identify product names associated with multiple SKU records.

```sql
SELECT
    name,
    COUNT(sku_id) AS number_of_skus
FROM zepto
GROUP BY name
HAVING COUNT(sku_id) > 1
ORDER BY number_of_skus DESC;
```

## Data Cleaning

### 7. Identify Products with Zero Prices

**Objective:** Find products with an MRP or discounted selling price of zero.

```sql
SELECT *
FROM zepto
WHERE mrp = 0
   OR discountedSellingPrice = 0;
```

### 8. Remove Products with Zero MRP

**Objective:** Remove records with zero MRP, following the cleaning rule used in this project.

```sql
DELETE FROM zepto
WHERE mrp = 0;
```

### 9. Convert Prices from Paise to Rupees

**Objective:** Convert MRP and discounted selling prices from paise to rupees.

```sql
UPDATE zepto
SET
    mrp = mrp / 100.0,
    discountedSellingPrice = discountedSellingPrice / 100.0;
```

Verify the converted prices:

```sql
SELECT
    mrp,
    discountedSellingPrice
FROM zepto;
```

**Important:** Run the conversion only once on the original paise-denominated data. Running it repeatedly will divide the prices again.

## Business Problems and Solutions

### 1. Find the Top 10 Best-Value Products by Discount Percentage

**Objective:** Identify products offering the highest discounts.

```sql
SELECT DISTINCT
    name,
    mrp,
    discountPercent
FROM zepto
ORDER BY discountPercent DESC
LIMIT 10;
```

### 2. Find High-MRP Products That Are Out of Stock

**Objective:** Identify products with an MRP above ₹300 that are currently out of stock.

```sql
SELECT DISTINCT
    name,
    mrp
FROM zepto
WHERE outOfStock = TRUE
  AND mrp > 300
ORDER BY mrp DESC;
```

### 3. Estimate Revenue by Product Category

**Objective:** Calculate estimated revenue for each category using discounted selling price and available inventory quantity.

```sql
SELECT
    category,
    SUM(discountedSellingPrice * availableQuantity) AS estimated_revenue
FROM zepto
GROUP BY category
ORDER BY estimated_revenue DESC;
```

*Note: This calculation estimates the value of available inventory at discounted prices. It is not actual realized sales revenue.*

### 4. Find Products with MRP Above ₹500 and Discounts Below 10%

**Objective:** Identify high-priced products offering relatively small discounts.

```sql
SELECT DISTINCT
    name,
    mrp,
    discountPercent
FROM zepto
WHERE mrp > 500
  AND discountPercent < 10
ORDER BY mrp DESC, discountPercent DESC;
```

### 5. Find the Top 5 Categories with the Highest Average Discount

**Objective:** Compare average discount percentages across product categories.

```sql
SELECT
    category,
    ROUND(AVG(discountPercent), 2) AS avg_discount
FROM zepto
GROUP BY category
ORDER BY avg_discount DESC
LIMIT 5;
```

### 6. Calculate Price per Gram for Products Weighing at Least 100 Grams

**Objective:** Compare product value based on discounted price per gram.

```sql
SELECT DISTINCT
    name,
    weightInGms,
    discountedSellingPrice,
    ROUND(
        discountedSellingPrice / NULLIF(weightInGms, 0),
        4
    ) AS price_per_gram
FROM zepto
WHERE weightInGms >= 100
ORDER BY price_per_gram ASC;
```

### 7. Classify Products by Weight

**Objective:** Group products into Low, Medium, and Bulk weight categories.

```sql
SELECT DISTINCT
    name,
    weightInGms,
    CASE
        WHEN weightInGms < 1000 THEN 'Low'
        WHEN weightInGms < 5000 THEN 'Medium'
        ELSE 'Bulk'
    END AS weight_category
FROM zepto
ORDER BY weightInGms;
```

The classification rules are:

- **Low:** Less than 1,000 grams
- **Medium:** 1,000 to 4,999 grams
- **Bulk:** 5,000 grams or more

### 8. Calculate Total Inventory Weight by Category

**Objective:** Measure the total weight of available inventory in each product category.

```sql
SELECT
    category,
    SUM(weightInGms * availableQuantity) AS total_inventory_weight
FROM zepto
GROUP BY category
ORDER BY total_inventory_weight DESC;
```

## Author

**Aravind K**  
Data Analyst | MIS Reporting | SQL | Power BI | Python  
Bengaluru, India


Thank you for checking out my project! Feedback and suggestions are welcome.
