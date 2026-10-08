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

## 🚀 Impact & Results
*Time Saved: Eliminated over 15 hours per week of manual data entry and formatting.
*Data Accuracy: Reduced human error in shift reporting to near zero by standardizing inputs programmatically.
*Real-time Analytics: Enabled the management team to query workforce distribution across all logistics centers without waiting for end-of-day manual reports.

## 💻 Sample Code Snippet (Sanitized)
Here is a conceptual snippet demonstrating the data transformation phase before loading it into the data warehouse:

```javascript
function doGet() {
  return HtmlService.createHtmlOutputFromFile('Index')
      .setTitle('Portal Cek Performa Bagger')
      .addMetaTag('viewport', 'width=device-width, initial-scale=1');
}

function getBaggerData(opsid, targetDate) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName("RAW DATA BAGGER");
  const data = sheet.getDataRange().getValues();
  const tz = ss.getSpreadsheetTimeZone(); // Mengambil zona waktu sheet agar tanggal tidak meleset

  let results = {};
  let baggerFullName = "";
  let searchId = opsid.toString().trim().toLowerCase();

  // Mematenkan posisi kolom sesuai gambarmu (A=0, B=1, C=2, dst)
  const idxDate = 0;   // Kolom A (Date)
  const idxBagger = 2; // Kolom C (Bagger)
  const idxTime = 5;   // Kolom F (Time)
  const idxQty = 6;    // Kolom G (Qty)
  const idxOpsId = 8;  // Kolom I (OPSID)

  // Langsung baca dari baris pertama (index 0)
  for (let i = 0; i < data.length; i++) {
    let row = data[i];
    
    // Ambil target OPSID di Kolom I
    let currentOpsId = String(row[idxOpsId] || "").trim().toLowerCase(); 

    if (currentOpsId === searchId && searchId !== "") {
      
      if (baggerFullName === "") {
        baggerFullName = String(row[idxBagger]); 
      }

      let dateVal = row[idxDate];
      let dateDisplay = ""; 
      let dateCompare = ""; 
      
      // Pastikan format tanggal cocok dengan kalender HTML (yyyy-MM-dd)
      if (Object.prototype.toString.call(dateVal) === '[object Date]') {
        dateDisplay = Utilities.formatDate(dateVal, tz, "dd-MMM-yyyy");
        dateCompare = Utilities.formatDate(dateVal, tz, "yyyy-MM-dd");
      } else {
        dateDisplay = String(dateVal);
        dateCompare = String(dateVal); 
      }

      // Proses Filter Tanggal
      if (targetDate && targetDate !== "") {
        if (dateCompare !== targetDate) {
          continue; 
        }
      }

      let timeStr = String(row[idxTime]); 
      let qty = Number(row[idxQty]) || 0;

      if (!results[dateDisplay]) {
        results[dateDisplay] = { total: 0, details: [] };
      }

      results[dateDisplay].total += qty;
      results[dateDisplay].details.push({ time: timeStr, qty: qty });
    }
  }

  return {
    name: baggerFullName,
    data: results
  };
}
