# US Household Income Data Cleaning & Exploratory Data Analysis Project

<p align="center">
  <img src="https://raw.githubusercontent.com/halyna2300/US_Household_Income_SQL_Project/main/US.Household.Income.jpg" width="500">
</p>

---

## 1. 📌 Project Overview
This project focuses on cleaning and analyzing a dataset related to U.S. household income using SQL. The goal was to gain insights into income distribution, land and water areas, place types, and geographic patterns across states and cities.

---

## 2. 🎯 Objectives
- Clean and standardize raw U.S. income and geographic data.
- Identify and remove duplicate records and formatting issues (like BOM characters).
- Normalize inconsistent or misspelled state and type values.
- Join and explore the data to extract income-based trends.
- Develop SQL queries to summarize and analyze land and water areas by location.

---

## 3. 🗂️ Dataset
The data for this project is sourced from two local CSV files:
- **USHouseholdIncome.csv** — Contains county-level geographic and demographic data including type, state, city, land/water area, and income values.
- **USHouseholdIncome_Statistics.csv** — Contains supplemental statistics for each record ID including mean and median income values.

📎 **Official Source:**  
U.S. Census Bureau – Historical Income Tables: Households  
🔗 [https://www.census.gov/data/tables/time-series/demo/income-poverty/historical-income-households.html](https://www.census.gov/data/tables/time-series/demo/income-poverty/historical-income-households.html)

---

## 4. 🛠️ Tools Used
- **SQL (MySQL)** — Data cleaning and analysis  
- **DBeaver / MySQL Workbench** — SQL execution interface  
- **GitHub** — Version control and documentation

---

## 5. 🧹 Data Cleaning Queries

### 5.1 Rename BOM Column

```sql
ALTER TABLE household_income_stat RENAME COLUMN `﻿id` TO `id`;
```

### 5.2 Remove Duplicate Records

```sql
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
```

### 5.3 Fix Typographical Errors in State Names

```sql
UPDATE household_income_stat 
SET State_Name = 'Georgia'
WHERE State_Name = 'georia';

UPDATE household_income_stat 
SET State_Name = 'Alabama'
WHERE State_Name = 'alabama';
```

### 5.4 Fix Place Based on County

```sql
UPDATE us_household_income
SET Place = 'Autaugaville'
WHERE City = 'Vinemont'
AND County = 'Autauga County';
```

### 5.5 Standardize Type Values

```sql
UPDATE us_household_income 
SET Type = 'Borough'
WHERE Type = 'Boroughs';
```

### 5.6 Handle Missing or Invalid Land Values

```sql
SELECT DISTINCT ALand
FROM us_household_income 
WHERE ALand = '' OR ALand = 0 OR ALand IS NULL;
```

---

## 6. Exploratory Data Analysis (EDA)

### 6.1 Count Total Records

```sql
SELECT COUNT(id) FROM household_income_stat;
```

### 6.2 Identify Duplicate IDs

```sql
SELECT id, COUNT(id)
FROM household_income_stat
GROUP BY id
HAVING COUNT(id) > 1;
```

### 6.3 Frequency of State Names

```sql
SELECT State_Name, COUNT(State_Name)
FROM household_income_stat
GROUP BY State_Name;
```

### 6.4 List Unique States

```sql
SELECT DISTINCT State_Name 
FROM household_income_stat
ORDER BY 1;
```

### 6.5 Type Distribution

```sql
SELECT Type, COUNT(Type)
FROM us_household_income
GROUP BY Type;
```

---

## 7. Data Analysis & Insights

### 7.1 Join Tables and Filter Non-Zero Mean

```sql
SELECT *
FROM us_household_income u
INNER JOIN household_income_stat us 
ON u.id = us.id
WHERE Mean <> 0;
```

### 7.2 Average Income by State

```sql
SELECT u.State_Name, ROUND(AVG(Mean), 1), ROUND(AVG(Median), 1)
FROM us_household_income u
INNER JOIN household_income_stat us 
ON u.id = us.id
WHERE Mean <> 0
GROUP BY u.State_Name
ORDER BY 3 ASC
LIMIT 10;
```

### 7.3 Average Income by Place Type

```sql
SELECT Type, COUNT(Type), ROUND(AVG(Mean), 1), ROUND(AVG(Median), 1)
FROM us_household_income u
INNER JOIN household_income_stat us 
ON u.id = us.id
WHERE Mean <> 0
GROUP BY 1
HAVING COUNT(Type) > 100
ORDER BY 4 DESC
LIMIT 20;
```

### 7.4 Top Cities by Average Income

```sql
SELECT u.State_Name, City, ROUND(AVG(Mean), 1), ROUND(AVG(Median), 1)
FROM us_household_income u
INNER JOIN household_income_stat us 
ON u.id = us.id
GROUP BY u.State_Name, City
ORDER BY ROUND(AVG(Mean), 1) DESC;
```

### 7.5 Sum of Land and Water by State

```sql
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
