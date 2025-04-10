# US Household Income Data Cleaning & Exploratory Data Analysis Project

<p align="center">
  <img src="https://github.com/halyna2300/US_Household_Income_SQL_Project/blob/main/US.Household.Income.jpg?raw=true" width="500">
</p>
---

## 📌 Project Overview
This project focuses on cleaning and analyzing a dataset related to U.S. household income using SQL. The goal was to gain insights into income distribution, land and water areas, place types, and geographic patterns across states and cities.

---

## 🎯 Objectives
- Clean and standardize raw U.S. income and geographic data.
- Identify and remove duplicate records and formatting issues (like BOM characters).
- Normalize inconsistent or misspelled state and type values.
- Join and explore the data to extract income-based trends.
- Develop SQL queries to summarize and analyze land and water areas by location.

---

## 🗂️ Dataset
The data for this project is sourced from two local CSV files:
- **USHouseholdIncome.csv** — Contains county-level geographic and demographic data including type, state, city, land/water area, and income values.
- **USHouseholdIncome_Statistics.csv** — Contains supplemental statistics for each record ID including mean and median income values.

📎 **Official Source:**  
U.S. Census Bureau – Historical Income Tables: Households  
🔗 [https://www.census.gov/data/tables/time-series/demo/income-poverty/historical-income-households.html](https://www.census.gov/data/tables/time-series/demo/income-poverty/historical-income-households.html)

---

## 🛠️ Tools Used
- **SQL (MySQL)** — Data cleaning and analysis  
- **DBeaver / MySQL Workbench** — SQL execution interface  
- **GitHub** — Version control and documentation

---

## 🧹 Data Cleaning Queries

```sql
-- Rename BOM-affected column
ALTER TABLE household_income_stat RENAME COLUMN `﻿id` TO `id`;

-- Remove duplicate IDs
DELETE FROM us_household_income
WHERE row_id IN (
    SELECT row_id
    FROM (
        SELECT row_id, id,
        ROW_NUMBER() OVER (PARTITION BY id ORDER BY id) AS row_num
        FROM us_household_income 
    ) row_table
    WHERE row_num > 1
);

-- Correct typos in state names
UPDATE household_income_stat 
SET State_Name = 'Georgia'
WHERE State_Name = 'georia';

UPDATE household_income_stat 
SET State_Name = 'Alabama'
WHERE State_Name = 'alabama';

-- Fix Place based on City & County
UPDATE us_household_income
SET Place = 'Autaugaville'
WHERE City = 'Vinemont'
AND County = 'Autauga County';

-- Standardize Type values
UPDATE us_household_income 
SET Type = 'Borough'
WHERE Type = 'Boroughs';

-- Check for invalid ALand values
SELECT DISTINCT ALand
FROM us_household_income 
WHERE ALand = '' OR ALand = 0 OR ALand IS NULL;
```

---

## 📊 EDA Queries

```sql
-- Count records
SELECT COUNT(id) FROM household_income_stat;

-- Check for duplicates
SELECT id, COUNT(id)
FROM household_income_stat
GROUP BY id
HAVING COUNT(id) > 1;

-- Frequency of State Names
SELECT State_Name, COUNT(State_Name)
FROM household_income_stat
GROUP BY State_Name;

-- Unique States
SELECT DISTINCT State_Name 
FROM household_income_stat
ORDER BY 1;

-- Type distribution
SELECT Type, COUNT(Type)
FROM us_household_income
GROUP BY Type;
```

---

## 📈 Data Analysis and Joins

```sql
-- Join both tables
SELECT *
FROM us_household_income u
INNER JOIN household_income_stat us 
ON u.id = us.id
WHERE Mean <> 0;

-- Average income by state
SELECT u.State_Name, ROUND(AVG(Mean), 1), ROUND(AVG(Median), 1)
FROM us_household_income u
INNER JOIN household_income_stat us 
ON u.id = us.id
WHERE Mean <> 0
GROUP BY u.State_Name
ORDER BY 3 ASC
LIMIT 10;

-- Average income by place type
SELECT Type, COUNT(Type), ROUND(AVG(Mean), 1), ROUND(AVG(Median), 1)
FROM us_household_income u
INNER JOIN household_income_stat us 
ON u.id = us.id
WHERE Mean <> 0
GROUP BY 1
HAVING COUNT(Type) > 100
ORDER BY 4 DESC
LIMIT 20;

-- Top income by city
SELECT u.State_Name, City, ROUND(AVG(Mean), 1), ROUND(AVG(Median), 1)
FROM us_household_income u
INNER JOIN household_income_stat us 
ON u.id = us.id
GROUP BY u.State_Name, City
ORDER BY ROUND(AVG(Mean), 1) DESC;

-- State-level land and water area
SELECT State_Name, SUM(ALand), SUM(AWater)
FROM us_household_income
GROUP BY State_Name
ORDER BY 3 DESC
LIMIT 10;
```

---

## 🧠 Conclusion
With the data now clean and explored, this analysis provides insights into income patterns, geographic distributions, and place-level statistics across the U.S. These SQL queries lay the foundation for further statistical analysis or data visualization.

---

## 📁 Folder Structure

```
📁 us-household-income-sql/
├── 📄 README.md
├── 📄 cleaning_queries.sql
├── 📄 eda_queries.sql
├── 📊 us_household_income.csv
├── 📊 household_income_stat.csv
```
