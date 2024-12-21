# ☕ Coffee Chain Analytics - SQL Documentation

<div align="center">

![MySQL](https://img.shields.io/badge/MySQL-8.0%2B-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Documentation](https://img.shields.io/badge/Documentation-Complete-success?style=for-the-badge)

</div>

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

## 📝 Documentation Standards

1. **Query Storage**: All SQL queries are saved for reference
2. **Collaboration**: Documentation ensures smooth team collaboration
3. **Results Capture**: Query results and screenshots are stored
4. **Client Requirements**: Documentation maintained for client proof requests
5. **Standards Compliance**: Follow project/company documentation standards

---
<div align="center">

**[View Project Repository](your-repo-link) • [Report Issues](your-issues-link)**

</div>
