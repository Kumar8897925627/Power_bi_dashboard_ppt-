# Power_bi_dashboard_ppt-
:  🚗 Car Price Analysis &amp; Dashboard using Python, SQL, and Power BI Complete end-to-end data analytics project on car pricing dataset from Kaggle. Includes data cleaning using Pandas, Exploratory Data Analysis using SQL (Oracle 11g) with group by, aggregations, HAVING, joins, and interactive Power BI dashboard featuring KPIs and visual insights.
# Car Price Prediction — Repository README + PPT Slide Text

> This document contains: a suggested GitHub repository layout, README content, code snippets (pandas cleaning + SQL EDA), and slide-by-slide text you can copy-paste directly into PowerPoint for a polished project presentation.

---

## Repository structure (suggested)

```
car-price-project/
├── data/
│   ├── raw/
│   │   └── car_price_prediction.csv
│   └── cleaned/
│       └── car_price_prediction_cleaned.csv
├── notebooks/
│   └── 01_data_cleaning.py
├── sql/
│   └── eda_queries.sql
├── powerbi/
│   ├── dashboard.pbix   # (optional - store backup/version)
│   └── powerbi_readme.md
├── ppt/
│   └── slides_text_for_powerpoint.md
├── README.md
└── LICENSE
```

---

## README.md (copy-paste into repository root)

````
# Car Price Analysis & Dashboard Project

This repository contains a small end-to-end project for exploring a car pricing dataset and building an interactive Power BI dashboard.

**Contents**
- `data/` — raw and cleaned CSV files
- `notebooks/01_data_cleaning.py` — pandas script used to clean and prepare the dataset
- `sql/eda_queries.sql` — collection of EDA SQL queries (GROUP BY, aggregations, HAVING, joins, outlier detection)
- `powerbi/` — Power BI assets and README describing visuals and layout
- `ppt/slides_text_for_powerpoint.md` — slide-by-slide text you can copy into PowerPoint

## How to reproduce

1. Clone the repo
```bash
git clone <your-repo-url>
cd car-price-project
````

2. Install Python dependencies (for cleaning script)

```bash
python -m venv venv
source venv/bin/activate   # or venv\Scripts\activate on Windows
pip install pandas openpyxl cx_Oracle
```

3. Run data cleaning script

```bash
python notebooks/01_data_cleaning.py
```

This generates `data/cleaned/car_price_prediction_cleaned.csv`.

4. Load the cleaned CSV into your Oracle database (SQL*Loader, SQL Developer or Python `cx_Oracle`).

5. Open the Power BI file in `powerbi/dashboard.pbix` or create the visuals following `powerbi/powerbi_readme.md`.

## License

Include your chosen license.

````

---

## notebooks/01_data_cleaning.py (copy-ready script)

```python
import pandas as pd

# 1. Load
df = pd.read_csv('data/raw/car_price_prediction.csv')

# 2. Basic cleaning examples
# Trim whitespace
for col in ['Brand','Fuel Type','Transmission','Condition','Model']:
    if col in df.columns:
        df[col] = df[col].astype(str).str.strip()

# Fix data types
df['Year'] = pd.to_numeric(df['Year'], errors='coerce').astype('Int64')
df['Engine Size'] = pd.to_numeric(df['Engine Size'], errors='coerce')
df['Mileage'] = pd.to_numeric(df['Mileage'], errors='coerce')
df['Price'] = pd.to_numeric(df['Price'], errors='coerce')

# Impute or drop
df['Mileage'].fillna(df['Mileage'].median(), inplace=True)
df['Engine Size'].fillna(df['Engine Size'].median(), inplace=True)
df['Price'].fillna(df['Price'].median(), inplace=True)

# Remove duplicates
df = df.drop_duplicates()

# Remove unrealistic values (example)
df = df[(df['Price'] > 100) & (df['Mileage'] >= 0)]

# Feature engineering
CURRENT_YEAR = 2025
if 'Year' in df.columns:
    df['Car_Age'] = CURRENT_YEAR - df['Year']

# Save cleaned file
df.to_csv('data/cleaned/car_price_prediction_cleaned.csv', index=False)
print('Saved cleaned dataset to data/cleaned/car_price_prediction_cleaned.csv')

# Optionally export a sample to preview
df.head(20).to_csv('data/cleaned/sample_preview.csv', index=False)
````

---

## sql/eda_queries.sql (copy-ready)

> Put this file into `sql/eda_queries.sql`. It's the same EDA script you used but cleaned for the repo.

```
/* 1. Basic Overview */
SELECT COUNT(*) AS total_records FROM CAR;

SELECT CAR_ID, COUNT(*) AS duplicate_count
FROM CAR
GROUP BY CAR_ID
HAVING COUNT(*) > 1;

SELECT
  SUM(CASE WHEN BRAND IS NULL THEN 1 ELSE 0 END) AS missing_brand,
  SUM(CASE WHEN YEAR IS NULL THEN 1 ELSE 0 END) AS missing_year,
  SUM(CASE WHEN ENGINE_SIZE IS NULL THEN 1 ELSE 0 END) AS missing_engine_size,
  SUM(CASE WHEN FUEL_TYPE IS NULL THEN 1 ELSE 0 END) AS missing_fuel_type,
  SUM(CASE WHEN TRANSMISSION IS NULL THEN 1 ELSE 0 END) AS missing_transmission,
  SUM(CASE WHEN MILEAGE IS NULL THEN 1 ELSE 0 END) AS missing_mileage,
  SUM(CASE WHEN CONDITION IS NULL THEN 1 ELSE 0 END) AS missing_condition,
  SUM(CASE WHEN PRICE IS NULL THEN 1 ELSE 0 END) AS missing_price
FROM CAR;

/* 2. Descriptive Stats */
SELECT MIN(PRICE) min_price, MAX(PRICE) max_price, ROUND(AVG(PRICE),2) avg_price FROM CAR;

SELECT BRAND, COUNT(*) total_cars, ROUND(AVG(PRICE),2) avg_price, MIN(PRICE) min_price, MAX(PRICE) max_price
FROM CAR
GROUP BY BRAND
ORDER BY avg_price DESC;

/* 3. Category-wise */
SELECT FUEL_TYPE, COUNT(*) total_cars, ROUND(AVG(PRICE),2) avg_price FROM CAR GROUP BY FUEL_TYPE;
SELECT TRANSMISSION, COUNT(*) total_cars, ROUND(AVG(PRICE),2) avg_price FROM CAR GROUP BY TRANSMISSION;
SELECT CONDITION, COUNT(*) total_cars, ROUND(AVG(PRICE),2) avg_price FROM CAR GROUP BY CONDITION ORDER BY avg_price DESC;

/* 4. Trend */
SELECT (2025 - YEAR) CAR_AGE, ROUND(AVG(PRICE),2) avg_price, COUNT(*) total_cars FROM CAR GROUP BY (2025 - YEAR) ORDER BY CAR_AGE;

/* 5. Top/N */
SELECT * FROM (SELECT BRAND, MODEL, PRICE, MILEAGE, YEAR FROM CAR ORDER BY PRICE DESC) WHERE ROWNUM <= 5;

/* 6. Outliers */
SELECT CAR_ID, BRAND, MODEL, PRICE FROM CAR WHERE PRICE > (SELECT AVG(PRICE) + 2 * STDDEV(PRICE) FROM CAR) OR PRICE < (SELECT AVG(PRICE) - 2 * STDDEV(PRICE) FROM CAR);
```

---

## powerbi/powerbi_readme.md (short guidance)

```
Power BI Dashboard Notes

Visuals created:
- 3 Cards: Total Cars, Average Price, Most Popular Brand
- Donut: Average Price by Model
- Clustered Bar: Average Price by Fuel Type
- Column Chart: Price by Brand (with Condition as legend)

Slicers used: Brand, Fuel Type, Transmission, Condition, Year

Theme & Design:
- Background image used (store only a small licensed image or link)
- Keep consistent colors and fonts

Export:
- Save an offline copy `dashboard.pbix` and place a small screenshot in `powerbi/screenshots/` for GitHub preview.
```

---

## ppt/slides_text_for_powerpoint.md (Slide-by-slide text)

> Copy these slide titles + bullet points directly into your PowerPoint slides. Place screenshots of the visuals where indicated.

```
Slide 1 — Title
Car Price Prediction Dashboard
Author: [Your Name]
Date: [Today’s Date]

Slide 2 — Project Overview
• Goal: Explore car pricing dataset and create an interactive dashboard in Power BI
• Data source: Kaggle (car_price_prediction.csv)
• Deliverables: Cleaned dataset, SQL EDA queries, Power BI dashboard

Slide 3 — Dataset Summary
• Rows: [replace with COUNT(*) result]
• Key columns: Car_ID, Brand, Model, Year, Engine Size, Fuel Type, Transmission, Mileage, Condition, Price
• Data issues handled: missing values, duplicates, incorrect types

Slide 4 — Data Cleaning (Pandas)
• Trimmed whitespace, fixed datatypes
• Imputed missing numeric values with median
• Removed duplicates and unrealistic records
• Engineered: Car_Age = 2025 - Year

Slide 5 — Cleaning Script (Key Snippets)
• `df['Mileage'].fillna(df['Mileage'].median(), inplace=True)`
• `df = df.drop_duplicates()`
• `df.to_csv('data/cleaned/car_price_prediction_cleaned.csv', index=False)`

Slide 6 — SQL EDA (Approach)
• Use SQL for group-by, aggregations, HAVING, joins and outlier detection
• Created view `V_CAR_EDA` to simplify analysis

Slide 7 — Important SQL Queries
• Total records: `SELECT COUNT(*) FROM CAR;`
• Avg price per brand: `SELECT BRAND, AVG(PRICE) FROM CAR GROUP BY BRAND;`
• Price outliers: `SELECT ... WHERE PRICE > (AVG+2*STDDEV)`

Slide 8 — Power BI Dashboard (Layout)
• Top: 3 Cards (Total Cars, Average Price, Most Popular Brand)
• Left: Slicers (Brand, Fuel Type, Transmission, Condition, Year)
• Main canvas: Donut (Avg Price by Model), Clustered Bar (Avg Price by Fuel), Column Chart (Price by Brand & Condition)

Slide 9 — Visuals: Donut (Avg price by Model)
• Purpose: Compare average price across models
• Screenshot: [Paste Power BI screenshot here]

Slide 10 — Visuals: Clustered Bar (Avg price by Fuel Type)
• Purpose: See how fuel type impacts price
• X-axis: Fuel Type | Y-axis: Average Price
• Screenshot: [Paste here]

Slide 11 — Visuals: Column Chart (Price by Brand & Condition)
• Purpose: Compare price distributions across brands and conditions
• X-axis: Brand | Y-axis: Price | Legend: Condition
• Screenshot: [Paste here]


