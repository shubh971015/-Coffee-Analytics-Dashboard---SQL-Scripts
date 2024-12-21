# ☕ Coffee Chain Analytics - SQL Documentation

# Coffee Sales Analysis Project

## Project Overview
This project involves analyzing sales data for a coffee chain. The analysis includes various SQL queries to gain insights into sales performance, customer buying patterns, and store performance. The project is implemented using MySQL.

## Getting Started

### Prerequisites
- MySQL Server
- MySQL Workbench
- 
## 📑 Table of Contents
- [Database Setup](#database-setup)
- [Data Preparation](#data-preparation)
- [Sales Analytics](#sales-analytics)
- [Store Performance](#store-performance)
- [Customer Insights](#customer-insights)
- [Time-Based Analysis](#time-based-analysis)
- [Product Analysis](#product-analysis)

## 🚀 Database Setup

### Database Creation
```sql
CREATE DATABASE COFFEE_PROJECT;
USE COFFEE_PROJECT;

-- Fix column name in COFFEE table
ALTER TABLE COFFEE
CHANGE ï»¿transaction_id transaction_id INT;
```

### Data Import
```sql
LOAD DATA INFILE 'D:\\practdatasets\\Sql_project\\Coffee_chain\\COFFEE_DATA\\COFFEE.CSV' 
INTO TABLE coffee 
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"' 
LINES TERMINATED BY '\n' 
IGNORE 1 LINES;

-- Disable safe updates for bulk operations
SET SQL_SAFE_UPDATES = 0;
```

## 📊 Data Preparation

### Date and Time Formatting
```sql
-- Update transaction_date format
UPDATE coffee
SET transaction_date = STR_TO_DATE(transaction_date, '%d-%m-%Y')
WHERE transaction_date IS NOT NULL;

-- Update transaction_time format
UPDATE coffee 
SET transaction_time = STR_TO_DATE(transaction_time, '%H:%i:%s') 
WHERE transaction_time IS NOT NULL;
```

## 📈 Sales Analytics

### Monthly Sales Performance
```sql
WITH Sales_analysis AS (
    SELECT 
        YEAR(Transaction_date) AS 'Year',
        MONTH(Transaction_date) AS 'Month',
        ROUND(SUM(transaction_qty * unit_price), 2) AS Total_sales,
        LAG(ROUND(SUM(transaction_qty * unit_price), 2)) 
            OVER (ORDER BY MONTH(Transaction_date)) AS previous_sales
    FROM coffee
    GROUP BY YEAR(Transaction_date), MONTH(Transaction_date)
)
SELECT 
    year,
    month,
    Total_sales,
    previous_sales,
    CONCAT(ROUND((Total_sales - previous_sales) / previous_sales * 100, 2), "%") 
        AS MoM_percet_change 
FROM Sales_analysis
WHERE month IN (4, 5);
```

### Quarterly Analysis
```sql
WITH Quarterly_sales AS (
    SELECT 
        QUARTER(transaction_date) AS 'Quarter',
        store_location,
        ROUND(SUM(transaction_qty * unit_price), 2) AS Total_sales,
        LAG(ROUND(SUM(transaction_qty * unit_price), 2)) 
            OVER (ORDER BY QUARTER(transaction_date)) AS previous_sales
    FROM coffee
    GROUP BY QUARTER(transaction_date), store_location
)
SELECT 
    Quarter,
    store_location,
    Total_sales,
    previous_sales,
    CONCAT(ROUND((Total_sales - previous_sales) / previous_sales * 100, 2), "%") 
        AS QoQ_percet_change 
FROM Quarterly_sales
WHERE Quarter = 2;
```

## 🏪 Store Performance

### Top Performing Stores
```sql
WITH store_sales AS ( 
    SELECT  
        MONTH(transaction_date) AS 'month',
        store_location,
        ROUND(SUM(transaction_qty * unit_price), 2) AS Total_sales
    FROM coffee
    GROUP BY MONTH(transaction_date), store_location 
)
SELECT  
    month, 
    store_location,
    Total_sales, 
    RANK() OVER(PARTITION BY store_location ORDER BY Total_sales DESC) AS store_rank 
FROM store_sales;
```

## 🕒 Time-Based Analysis

### Daily Performance Categories
```sql
WITH Daily_sales AS (
    SELECT
        DAY(transaction_date) AS days,
        ROUND(SUM(transaction_qty * unit_price), 2) AS total_sales
    FROM coffee
    WHERE MONTH(transaction_date) = 5
    GROUP BY transaction_date
)
SELECT 
    days,
    Total_sales,
    CASE 
        WHEN Total_sales > AVG(total_sales) OVER() THEN "Above Avg"
        ELSE "Below Avg"
    END AS category
FROM Daily_sales;
```

### Weekend vs Weekday Analysis
```sql
SELECT 
    CASE 
        WHEN DAYOFWEEK(transaction_date) IN (1, 7) THEN 'Weekend'
        ELSE 'Weekdays'
    END AS day_type,
    ROUND(SUM(transaction_qty * unit_price), 2) AS Total_sales
FROM coffee
WHERE MONTH(transaction_date) = 5 
GROUP BY 
    CASE 
        WHEN DAYOFWEEK(transaction_date) IN (1, 7) THEN 'Weekend'
        ELSE 'Weekdays'
    END;
```

## 📦 Product Analysis

### Top 10 Products
```sql
SELECT 
    product_type,
    product_category,
    ROUND(SUM(transaction_qty * unit_price), 2) AS total_sales
FROM coffee
WHERE 
    MONTH(transaction_date) = 5 
    AND product_category = "coffee"
GROUP BY 
    product_type
ORDER BY 
    total_sales DESC
LIMIT 10;
```

### Customer Buying Patterns
```sql
SELECT  
    transaction_id,
    COUNT(product_type),
    CASE 
        WHEN COUNT(product_type) > 1 THEN "Multiple product" 
        ELSE "Single" 
    END AS "Category"
FROM coffee 
GROUP BY transaction_id;
```

## 🛠️ Utility Procedures

### Data Retrieval
```sql
DELIMITER $$
CREATE PROCEDURE GetAllData()
BEGIN
    SELECT * FROM coffee;
END$$
DELIMITER ;
```


### Setup

1. **Create and Use the Database:**
    ```sql
    CREATE DATABASE coffee_chain;
    USE coffee_chain;
    ```

2. **Create Tables:**
    ```sql
    CREATE TABLE coffee (
        Area INT,
        Date DATE,
        Product VARCHAR(30),
        Product_Line VARCHAR(30),
        Product_Type VARCHAR(30),
        State VARCHAR(30),
        Territory VARCHAR(30),
        Type VARCHAR(30),
        Budget_Profit INT,
        Budget_Sales INT,
        Marketing INT,
        Profit INT,
        Sales INT,
        Total_Expenses INT
    );
    ```

3. **Load Data:**
    ```sql
    LOAD DATA INFILE 'E:\\ccc.csv'
    INTO TABLE coffee
    FIELDS TERMINATED BY ','
    ENCLOSED BY '"'
    IGNORE 1 ROWS;
    
    DESCRIBE coffee;
    ```

4. **Verify Data Load:**
    ```sql
    SELECT * FROM coffee_chain.coffee;
    ```

## Analysis Queries

### 1. Top 10 Areas by Total Sales of Espresso Products
```sql
SELECT area, SUM(sales) AS area_wise_sales, product_type 
FROM coffee
WHERE product_type = 'Espresso'
GROUP BY area
ORDER BY area_wise_sales DESC
LIMIT 10;
```

### 2. State in the East Market with the Lowest Profit for Espresso
```sql
SELECT area, state, product_type, Territory, SUM(profit) AS Total_profit
FROM coffee
WHERE product_type = 'Espresso' AND Territory = 'east'
GROUP BY area, Territory
ORDER BY Total_profit ASC
LIMIT 1;
```

### 3. Monthly Sales of Decaf Products Exceeding $30,000
```sql
SELECT area, type,
       EXTRACT(MONTH FROM Date) AS month_no,
       EXTRACT(YEAR FROM Date) AS year_no,
       SUM(sales) AS month_sales
FROM coffee
WHERE type = 'decaf'
GROUP BY month_no, year_no
HAVING month_sales > 30000
ORDER BY month_no, year_no;
```

### 4. State with the Highest Profit in the West Market in 2013
```sql
SELECT EXTRACT(YEAR FROM Date) AS year_no,
       state,
       SUM(profit) AS sum_profit,
       territory
FROM coffee
WHERE territory = 'west' AND EXTRACT(YEAR FROM Date) = 2013
GROUP BY state
ORDER BY sum_profit DESC;
```

### 5. Percentage (Expenses / Sales) of the State with the Lowest Profit
```sql
SELECT state, SUM(Total_Expenses), SUM(sales),
       CONCAT(ROUND((SUM(Total_Expenses) / SUM(sales)) * 100, 2), '%') AS percent_sales
FROM coffee
GROUP BY state
ORDER BY percent_sales ASC
LIMIT 5;
```

### 6. Highest Selling Product and State Combination
```sql
SELECT CONCAT(product_type, ' ', state) AS product_state_combined,
       SUM(sales) 
FROM coffee
GROUP BY product_type, state
ORDER BY SUM(sales) DESC 
LIMIT 3;
```

### 7. Contribution of Tea to the Overall Profit in 2012
```sql
SELECT CONCAT(ROUND(SUM(CASE WHEN product_type = 'tea' THEN profit ELSE 0 END) / SUM(profit) * 100, 2), '%') AS tea_contribution_percentage
FROM coffee
WHERE EXTRACT(YEAR FROM Date) = 2012;
```

### 8. Average % Profit / Sales for Products Starting with 'C'
```sql
SELECT CONCAT(ROUND(AVG(profit / sales) * 100, 2), '%') AS avg_profit_sales 
FROM coffee
WHERE product_type LIKE 'c%';
```

### 9. Distinct Count of Area Codes for the State with the Lowest Budget Margin in Small Market
```sql
SELECT state, COUNT(DISTINCT area) 
FROM coffee
GROUP BY state
ORDER BY COUNT(DISTINCT area) ASC
LIMIT 1;
```

### 10. Product Type without any Product in Top 5 Products by Sales
```sql
WITH ranked_products AS (
    SELECT Product_Type,
           ROW_NUMBER() OVER (PARTITION BY Product_Type ORDER BY SUM(Sales) DESC) AS row_num
    FROM coffee
    GROUP BY Product_Type
)
SELECT Product_Type
FROM ranked_products
WHERE row_num >= 5;
```

### 11. Top 5 Products by Sales for Each State
```sql
WITH ranked_products AS (
    SELECT Product,
           State,
           Sales,
           ROW_NUMBER() OVER (PARTITION BY State ORDER BY Sales DESC) AS row_num
    FROM coffee
)
SELECT State, Product, Sales
FROM ranked_products
WHERE row_num <= 5;
```

### 12. Top-Selling Products by Total Sales Revenue
```sql
SELECT product_type, SUM(sales) 
FROM coffee
GROUP BY product_type
ORDER BY SUM(sales) DESC
LIMIT 1;
```

### 13. Seasonal Trends in Sales Volume or Profit Margins
```sql
SELECT EXTRACT(MONTH FROM Date) AS month,
       EXTRACT(YEAR FROM Date) AS year,
       AVG(sales) AS avg_sales
FROM coffee
GROUP BY month
ORDER BY year, avg_sales ASC;
```

### 14. Profitability Across Different States or Territories
```sql
SELECT state, territory,
       EXTRACT(YEAR FROM Date) AS year,
       SUM(profit) 
FROM coffee
GROUP BY state, year
ORDER BY SUM(profit) DESC
LIMIT 5;
```

### 15. Average Profit Margin for Each Product Type
```sql
SELECT product_type, 
       AVG(profit / sales) * 100 AS avg_profit_margin
FROM coffee
GROUP BY product_type;
```

### 16. Identify Underperforming Products
```sql
SELECT product_type, 
       AVG(sales) AS avg_sales, 
       AVG(profit) AS avg_profit
FROM coffee
GROUP BY product_type
HAVING avg_sales < (SELECT AVG(sales) FROM coffee) 
   AND avg_profit < (SELECT AVG(profit) FROM coffee);
```



## 📝 Documentation Standards

1. **Query Storage**: All SQL queries are saved for reference
2. **Collaboration**: Documentation ensures smooth team collaboration
3. **Results Capture**: Query results and screenshots are stored
4. **Standards Compliance**: Follow project/company documentation standards

---
<div align="center">

**[View Project Repository](your-repo-link) • [Report Issues](your-issues-link)**

</div>
