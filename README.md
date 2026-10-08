# 🔄 Automated Data Pipeline: Google Sheets to Data Warehouse

> **Disclaimer:** *This repository contains a sanitized version of an automation script used for internal logistics operations. All credentials, database URIs, and sensitive company data have been removed or replaced with dummy variables to comply with confidentiality policies.*

## 📋 Project Overview
Developed a Python-based ETL (Extract, Transform, Load) pipeline to automate the migration of Daily Worker (DWR) schedules and logistics data from distributed Google Sheets into a centralized Data Warehouse. 

## 🎯 Problem Statement
Local logistics centers managed their daily shift schedules and worker allocations using standalone Google Sheets. This manual tracking created data silos, delayed reporting, and required warehouse administrators to manually clean and compile data before it could be queried or analyzed at the corporate level.

## 💡 The Solution
I built a Python automation script that runs as a scheduled cron job. The script performs the following workflow:
1. **Extract:** Connects to the Google Sheets API (using `gspread`) to fetch real-time schedule and operational data from multiple facility sheets.
2. **Transform:** Uses `pandas` to clean the data, handle missing values, standardize date formats, and enforce data types.
3. **Load:** Securely pushes the transformed, structured data into the relational Data Warehouse (using `SQLAlchemy`), making it immediately available for BI dashboards and operational queries.

## 🛠️ Tech Stack & Libraries
*   **Language:** Python 3.x
*   **Data Processing:** `pandas`, `numpy`
*   **APIs & Integrations:** `gspread`, `google-auth`
*   **Database Integration:** `SQLAlchemy` (PostgreSQL / MySQL)

## 💻 Sample Code Snippet (Sanitized)
Here is a conceptual snippet demonstrating the data transformation phase before loading it into the data warehouse:

```python
import pandas as pd
from sqlalchemy import create_engine

def transform_and_load(raw_data, table_name, db_connection_string):
    # Load raw Google Sheets data into a DataFrame
    df = pd.DataFrame(raw_data[1:], columns=raw_data[0])
    
    # Data Cleaning & Transformation
    df.dropna(subset=['Worker_ID', 'Shift_Date'], inplace=True)
    df['Shift_Date'] = pd.to_datetime(df['Shift_Date'], format='%Y-%m-%d')
    df['Status'] = df['Status'].str.strip().str.upper()
    
    # Establish Database Connection
    engine = create_engine(db_connection_string)
    
    # Load into Data Warehouse
    try:
        df.to_sql(table_name, con=engine, if_exists='append', index=False)
        print(f"Successfully loaded {len(df)} records into {table_name}.")
    except Exception as e:
        print(f"Database insertion failed: {e}")





