# Exploratory Data Analysis: Retail Sales

**Oasis Infobyte Internship (OIBSIP), Data Analytics, Level 1, Task 1**

This is an exploratory analysis of 1,000 retail transactions from 2023. It looks at sales trends, customer demographics and product-category performance, and ends with practical recommendations. The main finding is that **revenue is driven by price tier, not by basket size or demographics**: about a quarter of transactions (the high-ticket ones of $900 or more) bring in three quarters of revenue.

| | |
|---|---|
| **Dataset** | Retail Sales Dataset (Kaggle): 1,000 transactions × 9 columns |
| **Period** | 1 Jan 2023 – 1 Jan 2024 |
| **Techniques** | Data inspection, descriptive statistics, time-series aggregation, segmentation by age, gender and category, correlation analysis |
| **Tools** | Python, pandas, matplotlib, seaborn, Jupyter Notebook |
| **Total revenue** | $456,000 |

---

## 1. Objective

Use EDA to answer four business questions:
1. How do sales change month to month?
2. Who are the customers (age and gender)?
3. Which product categories drive revenue and volume?
4. What actually drives the value of a transaction?

## 2. Dataset

| Column | Type | Notes |
|---|---|---|
| Transaction ID | int | 1–1,000 |
| Date | date | 2023-01-01 to 2024-01-01 |
| Customer ID | string | **1,000 unique IDs, so every customer appears exactly once** |
| Gender | category | Female / Male |
| Age | int | 18–64 |
| Product Category | category | Beauty / Clothing / Electronics |
| Quantity | int | 1–4 |
| Price per Unit | int | Only 5 price points: $25, $30, $50 (low tier) and $300, $500 (premium tier) |
| Total Amount | int | Quantity × Price per Unit |

**Data quality:** 0 missing values and consistent data types. The only change needed was converting `Date` to a datetime.

## 3. Analysis Performed

- Data inspection (shape, dtypes, nulls) and descriptive statistics
- Monthly revenue trend (line chart)
- Customer age-group distribution (18–24, 25–34, 35–44, 45–54, 55–64)
- Gender split (pie chart)
- Revenue and units sold by product category
- Correlation heatmap (Age, Quantity, Price per Unit, Total Amount)
- Average spend by gender within each category

## 4. Key Findings

### 4.1 Transaction value is heavily right-skewed
| Statistic | Total Amount |
|---|---:|
| Mean | $456 |
| Median | $135 |
| 25th / 75th percentile | $60 / $900 |
| Min / Max | $25 / $2,000 |

The mean is **3.4× the median**. Transactions of **$900 or more make up 26% of orders but 75% of revenue**, so a small number of high-ticket purchases drives revenue and the typical basket does not.

### 4.2 Price, not quantity, drives transaction value
| Correlation with Total Amount | r |
|---|---:|
| Price per Unit | **0.85** |
| Quantity | 0.37 |
| Age | −0.06 |

Selling **higher-priced items** raises revenue far more than selling more units. Age has essentially no relationship with spend.

### 4.3 Monthly trend
| Peak | Low | Note |
|---|---|---|
| **May 2023: $53,150** | **September 2023: $23,620** | October rebounds to $46,580 |

Revenue swings by more than 2× between months, with no smooth seasonal curve. The **$1,530 for January 2024 is a single day** (the data ends on 1 Jan 2024), so it isn't a real decline and should be left out of trend reporting.

### 4.4 Customers
- **Gender:** 51% female (510) and 49% male (490). Revenue is split just as evenly ($232,840 vs $223,160).
- **Age:** the average is about 41. Every age band from 25 to 64 holds 20–23% of customers, and 18–24 holds 15%.
- **Spend by age:** average spend per transaction falls slightly with age, from **$501 (18–24) to $418 (55–64)**.

### 4.5 Product categories
| Category | Revenue | Share | Transactions | Units | Avg. transaction |
|---|---:|---:|---:|---:|---:|
| Electronics | $156,905 | 34.4% | 342 | 849 | $459 |
| Clothing | $155,580 | 34.1% | 351 | 894 | $443 |
| Beauty | $143,515 | 31.5% | 307 | 771 | $467 |

- No category dominates; the gap between the top and bottom category is only 3 percentage points.
- **Clothing sells the most units**, while **Beauty has the highest average transaction but the fewest transactions**. Beauty trails because of lower traffic, not smaller baskets.
- **Gender differences inside categories:** women spend more per Clothing transaction ($467 vs $420), and men spend more per Beauty transaction ($487 vs $451).

## 5. Recommendations

1. **Lift September with a targeted campaign.** It's the weakest month at less than half of May's revenue. A back-to-school or early-autumn promotion could close part of that gap.
2. **Protect and grow high-ticket sales.** About a quarter of orders bring in three quarters of revenue, so premium product availability, financing or instalment options, and upselling to higher price tiers matter more than volume discounts.
3. **Drive traffic to Beauty.** Beauty already converts to the highest basket value, so it needs more transactions, not bigger ones. Cross-category bundles (e.g. Clothing purchase → Beauty discount) use Clothing's high unit volume to bring shoppers to Beauty.
4. **Start capturing repeat behaviour.** Every customer in this dataset appears once, so loyalty, retention and lifetime value can't be measured. A loyalty ID at checkout would enable that analysis.

## 6. Limitations

- **Small sample:** 1,000 transactions, about 80 per month, so month-to-month swings may be partly noise.
- **No repeat customers**, product-level detail, store/channel or cost data, so margin and retention can't be analysed.
- The dataset looks synthetic (only 5 distinct prices, and the age and gender splits are very uniform). Patterns should be confirmed on real data before acting on them.

## 7. How to Reproduce

```bash
pip install pandas matplotlib seaborn jupyter
jupyter notebook EDA_Retail_Sales.ipynb
```

```
DataAnalytics-L1-EDARetailSales/
├── EDA_Retail_Sales.ipynb
└── retail_sales_dataset.csv
```

## Author

**Aarush Gupta**, Oasis Infobyte Summer Internship Program (Data Analytics Track)
