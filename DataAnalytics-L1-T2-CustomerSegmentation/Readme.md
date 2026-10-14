# Customer Segmentation: RFM and K-Means

**Oasis Infobyte Internship (OIBSIP), Data Analytics, Level 1, Task 2**

I segmented 4,338 customers of a UK online retailer into four behavioural groups, using **RFM (Recency, Frequency, Monetary)** features and **K-Means clustering**. The headline finding is strong revenue concentration: **5.2% of customers (VIP + Loyal) generate about 48% of revenue**, while a quarter of the base has effectively lapsed.

| | |
|---|---|
| **Dataset** | Online Retail (UCI / Kaggle): 541,909 invoice lines, Dec 2010 – Dec 2011 |
| **Techniques** | Data cleaning, RFM feature engineering, standardisation, Elbow method, K-Means, cluster profiling |
| **Tools** | Python, pandas, NumPy, scikit-learn (KMeans, StandardScaler), matplotlib, seaborn |
| **Headline result** | 4 segments: VIP (13 customers), Loyal (211), Casual (3,052), Lapsed (1,062) |

---

## 1. Business Question

**Which groups of customers behave differently, and how should marketing treat each group?** A single campaign for everyone wastes budget on customers who would buy anyway and under-invests in those about to churn.

## 2. Dataset

`onlineretail.csv` holds transaction-level data from a UK-based online gift retailer (about 89% of order lines are from the UK, with 37 countries in total).

| Column | Description |
|---|---|
| InvoiceNo | Invoice number. A prefix of `C` marks a cancellation |
| StockCode / Description | Product identifiers |
| Quantity, UnitPrice | Units and price per unit (£) |
| InvoiceDate | Timestamp (1 Dec 2010 – 9 Dec 2011) |
| CustomerID | Customer key |
| Country | Customer country |

## 3. Data Cleaning

| Step | Rows remaining |
|---|---:|
| Raw data | 541,909 |
| Removed rows with no `CustomerID` (135,080, which can't be attributed to a customer) | 406,829 |
| Removed cancellations (`InvoiceNo` starting with `C`), plus rows with quantity ≤ 0 or unit price ≤ 0 | **397,884** |

The cleaned data covers **4,338 unique customers**.

**Baseline customer metrics**

| Metric | Value |
|---|---:|
| Average order-line value | £22.40 |
| Average purchase frequency | 4.27 invoices per customer |
| Average customer lifetime value (total spend) | £2,054 |

## 4. Methodology

1. **RFM features per customer**
   - **Recency:** days since the last purchase, measured from a snapshot date one day after the last invoice.
   - **Frequency:** number of unique invoices.
   - **Monetary:** total spend (Quantity × UnitPrice).
2. **Scaling:** `StandardScaler`, so that Monetary (in the thousands) doesn't dominate Recency and Frequency.
3. **Choosing K:** Elbow method over K = 1–10. Inertia falls steeply up to K = 3–4 and flattens after that. I chose **K = 4** because it adds a business-meaningful split (Loyal vs VIP) without fragmenting the base.
4. **Clustering:** K-Means (`n_init=10`, `random_state=42`).
5. **Profiling:** mean R, F and M per cluster, plus scatter plots (Recency vs Monetary, Frequency vs Monetary) and cluster sizes.

## 5. Results: Segment Profiles

| Segment | Customers | % of customers | Avg Recency (days) | Avg Frequency (orders) | Avg Monetary (£) | % of revenue* |
|---|---:|---:|---:|---:|---:|---:|
| **VIP** (cluster 2) | 13 | 0.3% | 7 | 82.5 | 127,338 | 18.6% |
| **Loyal Regulars** (cluster 1) | 211 | 4.9% | 16 | 22.0 | 12,453 | 29.5% |
| **Casual** (cluster 3) | 3,052 | 70.4% | 44 | 3.7 | 1,350 | 46.2% |
| **Lapsed / At-risk** (cluster 0) | 1,062 | 24.5% | 249 | 1.6 | 478 | 5.7% |
| **Total** | **4,338** | 100% | | | | 100% (£8.91M) |

*Revenue share = segment size × average Monetary ÷ total customer spend.

## 6. Key Insights

- **Revenue is highly concentrated.** Just **224 customers (5.2%)**, the VIP and Loyal segments, account for **about 48% of revenue**. Losing even a few of them would hurt revenue noticeably.
- **VIPs behave like wholesalers.** They average 82 orders a year (more than one a week) and £127K each. This fits the retailer's known mix of trade buyers, so these accounts probably need account management more than consumer marketing.
- **The Casual segment is the volume engine.** It holds 70% of customers and 46% of revenue, but they buy only about 4 times a year. Small changes in their frequency affect revenue the most.
- **A quarter of the base has gone quiet.** Lapsed customers last bought **about 8 months ago** on average, ordered fewer than 2 times, and contribute under 6% of revenue.

## 7. Recommendations

| Segment | Goal | Action | KPI to track |
|---|---|---|---|
| VIP | **Retain** | Dedicated account contact, early access to new ranges, volume pricing instead of generic discounts | VIP churn, order frequency |
| Loyal | **Grow to VIP** | Tiered loyalty programme (points and tier thresholds), cross-sell and upsell bundles | Migration rate Loyal → VIP, average order value |
| Casual | **Increase frequency** | Monthly email and newsletter promotions, seasonal reminders, "you might also like" recommendations | Orders per customer per quarter |
| Lapsed | **Win back, cheaply** | A single "we miss you" incentive code. Suppress customers who don't respond to save marketing cost | Reactivation rate vs cost per reactivation |

## 8. Limitations and Next Steps

- **K-Means is sensitive to outliers.** Monetary is extremely right-skewed (one customer spent £77K on a single order), which is why the VIP cluster has only 13 members. A **log transform of F and M** before scaling would probably give more balanced clusters.
- **K was chosen by eye.** Silhouette scores or a gap statistic would make the choice more objective.
- **The segments are static.** Re-running monthly and tracking how customers move between segments (e.g. Loyal → Lapsed) would turn this into an early-warning churn signal.
- **B2B and B2C buyers are mixed together.** Separating wholesale accounts first would sharpen the consumer segments.
- Cluster numbers (0–3) depend on the random seed and library version, so read segments by their profile, not by their index.

## 9. How to Reproduce

```bash
pip install pandas numpy scikit-learn matplotlib seaborn jupyter
jupyter notebook Customer_Segmentation.ipynb
```

```
DataAnalytics-L1-T2-CustomerSegmentation/
├── Customer_Segmentation.ipynb
└── onlineretail.csv
```

## Author

**Aarush Gupta**, Oasis Infobyte Summer Internship Program (Data Analytics Track)
