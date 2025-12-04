
# COMP331 Final Project – Data Warehousing Quality Analysis

This repository contains the final project for **COMP 331 – Fall 2025**, completed by **Prabhjeevan Kaur**.  
The project evaluates data quality issues in a retail sales data warehouse using concepts from **Week 10 (Data Warehousing)**.

---

## 📌 Project Description

This project analyzes a retail sales dataset consisting of:

- **Sales fact table**
- **Products dimension**
- **Customers dimension**
- **Stores dimension**

The goals of this project are to:
- Identify data quality issues (completeness, validity, consistency, accuracy)
- Check referential integrity across the star schema
- Validate ETL workflow behaviour
- Examine Slowly Changing Dimension (SCD Type-2) handling
- Provide actionable improvements

The full analysis was performed using **Google Colab**.

---

## 📁 Repository Structure

```
COMP331-Final-Project/
│
├── README.md                 # Project description (THIS FILE)
│
├── data/                     # Raw and cleaned datasets (uploaded or download instructions)
│   ├── raw/
│   └── cleaned/
│
├── notebooks/                # Jupyter Colab / Python notebooks
│   └── analysis.ipynb
│
├── scripts/                  # ETL or validation scripts
│   └── etl_validation.py
│
└── results/                  # Output charts, tables, summary files
    ├── charts/
    ├── tables/
    └── summary.txt
```

---

## 📊 Included Analysis

The project includes:

### ✔ Completeness checks  
- Missing values in fact & dimension tables  
- Missing customer ID in sales fact table  

### ✔ Validity checks  
- Positive quantity & price  
- Data type correction for CustomerID  

### ✔ Consistency & Referential Integrity  
- Missing product keys (105, 106)  
- Missing customer key  
- Foreign key mismatch detection  

### ✔ Slowly Changing Dimensions (SCD Type-2)  
- Region change detection  
- Start/end date validation  

### ✔ Accuracy Checks  
- Recalculated totals vs ETL totals  

---

## 📈 Results

Outputs generated during analysis include:

- Bar charts and tables summarizing missing keys  
- SCD timelines  
- Summary statistics  
- Data validation logs
