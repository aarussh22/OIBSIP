# Data Cleaning: Cafe Sales Transactions

**Oasis Infobyte Internship (OIBSIP), Data Analytics, Level 1, Task 3**

I turned a deliberately messy file of 10,000 cafe transactions into an analysis-ready dataset. I recovered missing values from the business rules in the data itself instead of imputing them, kept **94.7% of rows** and ended with **0 nulls, 0 placeholder values and correct data types in every column**.

| | |
|---|---|
| **Dataset** | Dirty Cafe Sales (Kaggle): 10,000 rows × 8 columns |
| **Techniques** | Data-quality audit, placeholder detection, rule-based imputation, IQR outlier check, dtype correction |
| **Tools** | Python, pandas, NumPy, Jupyter Notebook |
| **Output** | `cleaned_cafe_sales.csv`: 9,469 rows × 8 columns |

---

## 1. Objective

Analyses are only as reliable as the data behind them. This task had three goals:
1. Find every data-quality problem in the raw file, including the ones that a plain `isnull()` check misses.
2. Fix each problem with a documented and defensible rule.
3. Keep as much real data as possible without inventing any.

## 2. Dataset

`dirty_cafe_sales.csv` holds 10,000 cafe transactions with 8 columns:

| Column | Expected type | Description |
|---|---|---|
| Transaction ID | string | Unique transaction key |
| Item | category | One of 8 menu items (Coffee, Tea, Cake, Cookie, Juice, Salad, Sandwich, Smoothie) |
| Quantity | integer | Units bought (1–5) |
| Price Per Unit | float | Menu price (1.0–5.0) |
| Total Spent | float | Quantity × Price Per Unit |
| Payment Method | category | Cash / Credit Card / Digital Wallet |
| Location | category | In-store / Takeaway |
| Transaction Date | datetime | Date of sale (2023) |

## 3. Data Quality Audit (Before Cleaning)

The file hides most of its problems. Besides real nulls, it uses the text placeholders `ERROR` and `UNKNOWN`, and `isnull()` doesn't count those.

| Column | Nulls | ERROR / UNKNOWN | Total missing | % missing |
|---|---:|---:|---:|---:|
| Item | 333 | 636 | 969 | 9.7% |
| Quantity | 138 | 341 | 479 | 4.8% |
| Price Per Unit | 179 | 354 | 533 | 5.3% |
| Total Spent | 173 | 329 | 502 | 5.0% |
| Payment Method | 2,579 | 599 | 3,178 | **31.8%** |
| Location | 3,265 | 696 | 3,961 | **39.6%** |
| Transaction Date | 159 | 301 | 460 | 4.6% |
| **Total** | **6,826** | **3,256** | **10,082** | |

**Audit findings**
- **32% of the missing values were hidden** (3,256 of 10,082). A report based only on `isnull()` would have understated the problem by about a third.
- **Wrong data types:** Quantity, Price Per Unit, Total Spent and Transaction Date were stored as text because of the placeholders, so none of the 4 numeric or date columns could be used as they were.
- **Duplicates:** 0 duplicate rows and 0 duplicate Transaction IDs.

## 4. Cleaning Approach

| Step | Action | Rationale |
|---|---|---|
| 1. Standardise | Stripped whitespace and converted `ERROR`/`UNKNOWN` to `NaN`. Coerced the numeric and date columns. | This puts every missing value into one form that the tools can detect. |
| 2. Recover **Price Per Unit** | Filled it from the item's menu price, using a mapping built from complete rows (e.g. Coffee 2.0, Cake 3.0, Tea 1.5). | Each item has exactly one price in the data, so the value is recovered from the data and not estimated. |
| 3. Recover **Quantity** | `Total Spent ÷ Price Per Unit` (**456 values recovered**) | `Total = Qty × Price` holds on every complete row. |
| 4. Recover **Total Spent** | `Quantity × Price Per Unit` | Same identity. |
| 5. Drop unrecoverable numeric rows | **74 rows** dropped | These rows were missing 2 of the 3 linked values, so nothing was left to rebuild them from. |
| 6. Drop rows missing **Transaction Date** | **457 rows** dropped (4.6%) | No other column holds the date, and inventing dates would distort any time-series analysis. |
| 7. Categorical gaps | Filled Item, Payment Method and Location with an explicit `"Unknown"` | Dropping them would remove up to 40% of rows. An explicit label keeps the rows and keeps the gap visible in later analysis. |
| 8. Duplicates | Checked again after cleaning: 0 removed | |
| 9. Outliers (IQR) | Quantity: 0 and Price: 0. Total Spent: 259 flagged, **all kept** | See below. |
| 10. Dtypes | Quantity → int, Price/Total → float, Date → datetime, text → category | |

**Outlier decision.** The IQR rule (upper fence 24.0) flagged 259 rows in Total Spent. Every one of them equals **exactly 25.0**, which is the largest possible order (5 units × the top menu price of 5.0). These are real large orders, not errors. Removing them would have biased revenue figures downward.

## 5. Results: Before vs After

| Metric | Before | After |
|---|---:|---:|
| Rows | 10,000 | **9,469** (94.7% retained) |
| Null values | 6,826 | **0** |
| ERROR / UNKNOWN placeholders | 3,256 | **0** |
| Duplicate rows | 0 | 0 |
| Numeric/date columns with correct dtype | 0 / 4 | **4 / 4** |

Only **531 rows (5.3%)** were removed, and each removal has a documented reason.

## 6. Key Takeaways

- **Audit beyond `isnull()`.** In this file, sentinel strings such as `ERROR` and `UNKNOWN` made up about a third of the missing data.
- **Use the business rules before imputing.** A fixed menu price and the `Total = Qty × Price` identity recovered hundreds of values exactly, with no mean or median guessing.
- **An outlier is not automatically an error.** The 259 flagged totals are the legitimate upper limit of the data.
- **Make missingness visible.** The `"Unknown"` category keeps the row count honest and lets later analyses decide how to treat it.

## 7. Limitations

- About 40% of Location and 32% of Payment Method values are `"Unknown"`. Any channel or payment-mix analysis built on this data should report results with and without that category.
- The price-to-item mapping assumes prices did not change during 2023. That holds in this dataset (one price per item) but might not hold for a real cafe.

## 8. How to Reproduce

```bash
pip install pandas numpy jupyter
jupyter notebook Cleaning_Data.ipynb
```

```
DataAnalytics-L1-T3-Cleaning/
├── Cleaning_Data.ipynb      # audit → cleaning → before/after report
├── dirty_cafe_sales.csv     # raw input
└── cleaned_cafe_sales.csv   # cleaned output (9,469 rows)
```

## Author

**Aarush Gupta**, Oasis Infobyte Summer Internship Program (Data Analytics Track)
