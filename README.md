
 DATA WAREHOUSING QUALITY ANALYSIS (GOOGLE COLAB)
# DATA WAREHOUSING QUALITY ANALYSIS (GOOGLE COLAB)

# Install libs (Colab usually has these, but this ensures availability)
!pip install pandas numpy matplotlib seaborn --quiet

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from google.colab import files

# Prompt user to upload the three CSV files
print("Upload your 3 dataset files: (Sales, Features, Stores)")
uploaded = files.upload()

# Load datasets (use the uploaded filenames or adjust if names differ)
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


 2. REFERENTIAL INTEGRITY (KEY QUALITY)


missing_in_stores_from_sales = set(sales["Store"]) - set(stores["Store"])
missing_in_stores_from_features = set(features["Store"]) - set(stores["Store"])

print("\nStore IDs in SALES not found in STORES:", missing_in_stores_from_sales)
print("Store IDs in FEATURES not found in STORES:", missing_in_stores_from_features)

# DATE alignment: coerce to datetime safely
sales["Date"] = pd.to_datetime(sales["Date"], errors='coerce')
features["Date"] = pd.to_datetime(features["Date"], errors='coerce')

date_mismatch = set(sales["Date"].dropna()) - set(features["Date"].dropna())
print("\nDates missing in Features (but present in Sales): sample 20")
print(list(date_mismatch)[:20])


3. COMPLETENESS CHECK

print("\n=== COMPLETENESS CHECK ===")

missing_sales = sales.isna().sum()
missing_features = features.isna().sum()
missing_stores = stores.isna().sum()

print("\nMissing values in Sales:\n", missing_sales)
print("\nMissing values in Features:\n", missing_features)
print("\nMissing values in Stores:\n", missing_stores)

# Export missing summary (align indices)
missing_summary = pd.DataFrame({
    "Sales": missing_sales,
    "Features": missing_features.reindex(missing_sales.index, fill_value=0),
    "Stores": missing_stores.reindex(missing_sales.index, fill_value=0)
})
missing_summary.to_csv("missing_summary.csv", index=True)


 4. CONSISTENCY CHECK

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

# Check object columns in features for potential format problems
wrong_types = features.select_dtypes(include="object").columns
print("\nColumns with potential formatting inconsistencies:", list(wrong_types))


 5. VALIDITY CHECK (BUSINESS RULES)

print("\n=== VALIDITY CHECK ===")

# Weekly_Sales must be >= 0
invalid_sales = sales[sales["Weekly_Sales"] < 0]
invalid_sales.to_csv("invalid_sales.csv", index=False)
print("Negative sales rows exported:", invalid_sales.shape[0])

# MarkDown outliers
markdown_cols = [c for c in features.columns if "MarkDown" in c]
for col in markdown_cols:
    threshold = features[col].quantile(0.99)
    outliers = features[features[col] > threshold]
    print(f"Outliers in {col} (>{threshold}):", outliers.shape[0])


 6. ACCURACY CHECK (OUTLIERS)

print("\n=== ACCURACY CHECK ===")

sales_Q1 = sales["Weekly_Sales"].quantile(0.25)
sales_Q3 = sales["Weekly_Sales"].quantile(0.75)
IQR = sales_Q3 - sales_Q1

outliers_sales = sales[(sales["Weekly_Sales"] < (sales_Q1 - 1.5 * IQR)) |
                       (sales["Weekly_Sales"] > (sales_Q3 + 1.5 * IQR))]

print("Weekly Sales Outliers:", outliers_sales.shape[0])
outliers_sales.to_csv("weekly_sales_outliers.csv", index=False)

# Plot boxplot (will display inline in Colab)
plt.figure(figsize=(10,4))
sns.boxplot(x=sales["Weekly_Sales"])
plt.title("Weekly Sales Outliers (Accuracy Check)")
plt.show()


 7. SLOWLY CHANGING DIMENSION (SCD) CHECK

print("\n=== SCD CHECK (Store Dimension Stability) ===")

scd_violations = stores.groupby("Store").agg({"Type": "nunique", "Size": "nunique"})
print(scd_violations)

violating_stores = scd_violations[(scd_violations["Type"] > 1) | (scd_violations["Size"] > 1)]
print("\nStores with unexpected SCD changes (should not change):")
print(violating_stores)


 8. MERGE CHECK — FINAL STAR SCHEMA VALIDATION

merged = sales.merge(features, on=["Store", "Date"], how="left")
merged = merged.merge(stores, on="Store", how="left")

missing_after_merge = merged.isna().sum()
missing_after_merge.to_csv("missing_after_merge.csv", index=True)

print("\n=== MERGED TABLE SHAPE ===")
print(merged.shape)

print("\nMissing values after merging (indicates ETL or dimension problems):")
print(missing_after_merge.head(20))

merged.to_csv("merged_fact_table.csv", index=False)

print("\nAll analysis completed. Download CSVs from the left panel (Files).")
