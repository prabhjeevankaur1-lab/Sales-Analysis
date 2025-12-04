
 DATA WAREHOUSING QUALITY ANALYSIS (GOOGLE COLAB)


!pip install pandas numpy matplotlib seaborn --quiet

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from google.colab import files

print("Upload your 3 dataset files: (Sales, Features, Stores)")
uploaded = files.upload()

 Load datasets
sales = pd.read_csv("sales data-set.csv")
features = pd.read_csv("Features data set.csv")
stores = pd.read_csv("stores data-set.csv")

print("\nFiles loaded successfully!")


 1. BASIC STRUCTURE (STAR SCHEMA CHECK)


print("\n=== Dataset Shapes ===")
print("Sales (Fact Table):", sales.shape)
print("Features (Dimension-like Table):", features.shape)
print("Stores (Dimension Table):", stores.shape)

print("\n=== Columns in Each Dataset ===")
print("Sales:", sales.columns.tolist())
print("Features:", features.columns.tolist())
print("Stores:", stores.columns.tolist())


 2. CHECK REFERENTIAL INTEGRITY (KEY QUALITY)


print("\n=== REFERENTIAL INTEGRITY CHECK ===")

# Stores referenced in Sales but not in Stores dimension
missing_in_stores_from_sales = set(sales["Store"]) - set(stores["Store"])
missing_in_stores_from_features = set(features["Store"]) - set(stores["Store"])

print("\nStore IDs in SALES not found in STORES:", missing_in_stores_from_sales)
print("Store IDs in FEATURES not found in STORES:", missing_in_stores_from_features)

# DATE key alignment
sales["Date"] = pd.to_datetime(sales["Date"], errors='coerce')
features["Date"] = pd.to_datetime(features["Date"], errors='coerce')

date_mismatch = set(sales["Date"]) - set(features["Date"])
print("\nDates missing in Features (but present in Sales): sample 20")
print(list(date_mismatch)[:20])

 3. COMPLETENESS CHECK (WAREHOUSE LOADING ISSUES)

print("\n=== COMPLETENESS CHECK ===")

missing_sales = sales.isna().sum()
missing_features = features.isna().sum()
missing_stores = stores.isna().sum()

print("\nMissing values in Sales:\n", missing_sales)
print("\nMissing values in Features:\n", missing_features)
print("\nMissing values in Stores:\n", missing_stores)

 Export missing summary
missing_summary = pd.DataFrame({
    "Sales": missing_sales,
    "Features": missing_features.reindex(missing_sales.index, fill_value=0),
    "Stores": missing_stores.reindex(missing_sales.index, fill_value=0)
})
missing_summary.to_csv("missing_summary.csv")

 4. CONSISTENCY CHECK (ETL TRANSFORMATION ISSUES)

print("\n=== CONSISTENCY CHECK ===")

# Duplicate PK checks
sales_dupes = sales.duplicated(subset=["Store", "Date", "Dept"]).sum()
features_dupes = features.duplicated(subset=["Store", "Date"]).sum()
stores_dupes = stores.duplicated(subset=["Store"]).sum()

print("Duplicate PK rows in Sales:", sales_dupes)
print("Duplicate PK rows in Features:", features_dupes)
print("Duplicate PK rows in Stores:", stores_dupes)

# Numeric columns consistency
invalid_neg_sales = sales[sales["Weekly_Sales"] < 0].shape[0]
print("\nInvalid negative Weekly_Sales:", invalid_neg_sales)

# Feature mismatches
wrong_types = features.select_dtypes(include="object").columns
print("\nColumns with potential formatting inconsistencies:", list(wrong_types))

5. VALIDITY CHECK (BUSINESS RULES)

print("\n=== VALIDITY CHECK ===")

# Weekly_Sales must be >= 0
invalid_sales = sales[sales["Weekly_Sales"] < 0]
invalid_sales.to_csv("invalid_sales.csv")

print("Negative sales found:", invalid_sales.shape[0])

# MarkDown fields must not be extreme
markdown_cols = [c for c in features.columns if "MarkDown" in c]

for col in markdown_cols:
    outliers = features[features[col] > features[col].quantile(0.99)]
    print(f"Outliers in {col}:", outliers.shape[0])


 6. ACCURACY CHECK (OUTLIERS + SUSPICIOUS RANGES)


print("\n=== ACCURACY CHECK ===")

sales_Q1 = sales["Weekly_Sales"].quantile(0.25)
sales_Q3 = sales["Weekly_Sales"].quantile(0.75)
IQR = sales_Q3 - sales_Q1

outliers_sales = sales[(sales["Weekly_Sales"] < sales_Q1 - 1.5*IQR) |
                       (sales["Weekly_Sales"] > sales_Q3 + 1.5*IQR)]

print("Weekly Sales Outliers:", outliers_sales.shape[0])

plt.figure(figsize=(10,4))
sns.boxplot(x=sales["Weekly_Sales"])
plt.title("Weekly Sales Outliers (Accuracy Check)")
plt.show()

 7. SLOWLY CHANGING DIMENSION (SCD) CHECK

print("\n=== SCD CHECK (Store Dimension Stability) ===")

# Check if Store size or type changed unexpectedly (should be stable)
scd_violations = stores.groupby("Store").agg({"Type": "nunique", "Size": "nunique"})
print(scd_violations)

violating_stores = scd_violations[(scd_violations["Type"] > 1) | (scd_violations["Size"] > 1)]
print("\nStores with unexpected SCD changes (should not change):")
print(violating_stores)


 8. MERGE CHECK — FINAL STAR SCHEMA VALIDATION


merged = sales.merge(features, on=["Store", "Date"], how="left")
merged = merged.merge(stores, on="Store", how="left")

missing_after_merge = merged.isna().sum()
missing_after_merge.to_csv("missing_after_merge.csv")

print("\n=== MERGED TABLE SHAPE ===")
print(merged.shape)

print("\nMissing values after merging (indicates ETL or dimension problems):")
print(missing_after_merge.head(20))

merged.to_csv("merged_fact_table.csv", index=False)

print("\nAll analysis completed. Download CSVs from the left panel.")




Upload your 3 dataset files: (Sales, Features, Stores)
•  Features data set.csv(text/csv) - 600478 bytes, last modified: n/a - 100% done
•  sales data-set.csv(text/csv) - 13264115 bytes, last modified: n/a - 100% done
•  stores data-set.csv(text/csv) - 577 bytes, last modified: n/a - 100% done
Saving Features data set.csv to Features data set (1).csv
Saving sales data-set.csv to sales data-set (1).csv
Saving stores data-set.csv to stores data-set (1).csv

Files loaded successfully!

=== Dataset Shapes ===
Sales (Fact Table): (421570, 5)
Features (Dimension-like Table): (8190, 12)
Stores (Dimension Table): (45, 3)

=== Columns in Each Dataset ===
Sales: ['Store', 'Dept', 'Date', 'Weekly_Sales', 'IsHoliday']
Features: ['Store', 'Date', 'Temperature', 'Fuel_Price', 'MarkDown1', 'MarkDown2', 'MarkDown3', 'MarkDown4', 'MarkDown5', 'CPI', 'Unemployment', 'IsHoliday']
Stores: ['Store', 'Type', 'Size']

=== REFERENTIAL INTEGRITY CHECK ===

Store IDs in SALES not found in STORES: set()
Store IDs in FEATURES not found in STORES: set()

Dates missing in Features (but present in Sales): sample 20
[]

=== COMPLETENESS CHECK ===

Missing values in Sales:
 Store                0
Dept                 0
Date            253414
Weekly_Sales         0
IsHoliday            0
dtype: int64

Missing values in Features:
 Store              0
Date            4905
Temperature        0
Fuel_Price         0
MarkDown1       4158
MarkDown2       5269
MarkDown3       4577
MarkDown4       4726
MarkDown5       4140
CPI              585
Unemployment     585
IsHoliday          0
dtype: int64

Missing values in Stores:
 Store    0
Type     0
Size     0
dtype: int64

=== CONSISTENCY CHECK ===
Duplicate PK rows in Sales: 250110
Duplicate PK rows in Features: 4860
Duplicate PK rows in Stores: 0

Invalid negative Weekly_Sales: 1285

Columns with potential formatting inconsistencies: []

=== VALIDITY CHECK ===
Negative sales found: 1285
Outliers in MarkDown1: 41
Outliers in MarkDown2: 30
Outliers in MarkDown3: 37
Outliers in MarkDown4: 35
Outliers in MarkDown5: 41

=== ACCURACY CHECK ===
Weekly Sales Outliers: 35521
 

=== SCD CHECK (Store Dimension Stability) ===
       Type  Size
Store            
1         1     1
2         1     1
3         1     1
4         1     1
5         1     1
6         1     1
7         1     1
8         1     1
9         1     1
10        1     1
11        1     1
12        1     1
13        1     1
14        1     1
15        1     1
16        1     1
17        1     1
18        1     1
19        1     1
20        1     1
21        1     1
22        1     1
23        1     1
24        1     1
25        1     1
26        1     1
27        1     1
28        1     1
29        1     1
30        1     1
31        1     1
32        1     1
33        1     1
34        1     1
35        1     1
36        1     1
37        1     1
38        1     1
39        1     1
40        1     1
41        1     1
42        1     1
43        1     1
44        1     1
45        1     1

Stores with unexpected SCD changes (should not change):
Empty DataFrame
Columns: [Type, Size]
Index: []

=== MERGED TABLE SHAPE ===
(27790282, 17)

Missing values after merging (indicates ETL or dimension problems):
Store                  0
Dept                   0
Date            27622126
Weekly_Sales           0
IsHoliday_x            0
Temperature            0
Fuel_Price             0
MarkDown1       13838098
MarkDown2       17480618
MarkDown3       15504138
MarkDown4       15372156
MarkDown5       13795997
CPI              2027312
Unemployment     2027312
IsHoliday_y            0
Type                   0
Size                   0
dtype: int64

All analysis completed. Download CSVs from the left panel.

