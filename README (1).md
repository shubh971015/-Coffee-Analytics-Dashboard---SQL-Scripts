
# ☕ Coffee Chain Analytics Project

![MySQL](https://img.shields.io/badge/MySQL-8.0%2B-blue)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![License](https://img.shields.io/badge/License-MIT-yellow)

> A comprehensive SQL analytics solution providing deep insights into coffee chain operations, sales patterns, and customer behavior.

## 📑 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Setup](#-setup)
- [Analysis](#-analysis)
- [Documentation](#-documentation)
- [Optimization](#-optimization)

## 🎯 Overview

This project provides end-to-end SQL analytics for coffee chain data, focusing on:

- Sales pattern analysis
- Store performance metrics
- Customer behavior insights
- Time-based trend analysis

## ✨ Features

- **Sales Analytics**: Month-over-month comparison
- **Store Performance**: Location-based analysis
- **Time Patterns**: Hourly and daily sales tracking
- **Customer Insights**: Purchase behavior analysis

## 🚀 Setup

### Database Initialization
```sql
CREATE DATABASE COFFEE_PROJECT;
USE COFFEE_PROJECT;

-- Table Creation
CREATE TABLE coffee (
    transaction_id INT,
    transaction_date DATE,
    transaction_time TIME,
    transaction_qty INT,
    unit_price DECIMAL(10,2),
    product_type VARCHAR(50),
    product_category VARCHAR(50),
    store_location VARCHAR(50)
);
```

### Data Import and Preparation
```sql
-- Import Data
LOAD DATA INFILE 'path_to_coffee.csv'
INTO TABLE coffee 
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"' 
LINES TERMINATED BY '\n' 
IGNORE 1 LINES;

-- Standardize Dates
UPDATE coffee
SET transaction_date = STR_TO_DATE(transaction_date, '%d-%m-%Y')
WHERE transaction_date IS NOT NULL;
```

## 📊 Analysis

### 1️⃣ Sales Performance
```sql
WITH Sales_analysis AS (
    SELECT 
        YEAR(Transaction_date) 'Year',
        MONTH(Transaction_date) 'Month',
        ROUND(SUM(transaction_qty*unit_price), 2) AS Total_sales,
        LAG(ROUND(SUM(transaction_qty*unit_price), 2)) 
            OVER(ORDER BY MONTH(Transaction_date)) AS previous_sales
    FROM coffee
    GROUP BY 
        YEAR(Transaction_date),
        MONTH(Transaction_date)
)
SELECT 
    year,
    month,
    Total_sales,
    previous_sales,
    CONCAT(ROUND((Total_sales-previous_sales)/previous_sales * 100,2),"%") 
        AS MoM_percent_change 
FROM Sales_analysis
WHERE month IN (4,5);
```

### 2️⃣ Store Performance
```sql
WITH store_sales AS (
    SELECT  
        MONTH(transaction_date) AS 'month',
        store_location,
        ROUND(SUM(transaction_qty*unit_price), 2) AS Total_sales 
    FROM coffee
    GROUP BY 
        MONTH(transaction_date),
        store_location 
)
SELECT 
    month, 
    store_location,
    Total_sales,
    RANK() OVER(PARTITION BY month 
        ORDER BY Total_sales DESC) AS store_rank
FROM store_sales
WHERE store_rank <= 2;
```

## 📝 Documentation Standards

### Required Practices
1. ✅ Save all executed SQL queries
2. 📋 Maintain collaboration documentation
3. 📸 Store query results and screenshots
4. 📄 Keep client proof requests
5. 📚 Follow project standards

## ⚡ Performance Tips

### Query Optimization
- Use appropriate indexes
- Optimize complex queries
- Regular maintenance schedules
- Performance monitoring

### Safety Measures
```sql
-- Create backup
CREATE TABLE backup_table AS 
SELECT * FROM coffee;

-- Create stored procedure
DELIMITER $$
CREATE PROCEDURE GetAllData()
BEGIN
    SELECT * FROM coffee;
END$$
DELIMITER ;
```

## 🛠️ Maintenance

### Daily Checks
- [ ] Data validation
- [ ] Backup verification
- [ ] Performance monitoring
- [ ] Error log review

### Monthly Tasks
- [ ] Index optimization
- [ ] Storage cleanup
- [ ] Performance analysis
- [ ] Documentation update

## 📈 Key Insights

### Sales Patterns
- Weekend vs Weekday performance analysis
- Peak hours identification
- Monthly sales trends identification

### Product Performance
```sql
SELECT 
    product_type,
    product_category,
    ROUND(SUM(transaction_qty*unit_price), 2) AS total_sales
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

## 📌 Important Notes

- Ensure date format consistency
- Validate all data imports
- Maintain regular backups
- Monitor storage usage

---

### 📫 Contributing
Feel free to submit issues and enhancement requests!

### 📝 License
This project is licensed under the MIT License - see the LICENSE file for details.