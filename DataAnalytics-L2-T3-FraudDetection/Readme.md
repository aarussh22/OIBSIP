# Credit Card Fraud Detection: Imbalanced Classification

**Oasis Infobyte Internship (OIBSIP), Data Analytics, Level 2, Task 3**

I detect fraudulent card transactions in a dataset where **only 1.51% of transactions are fraud**, and compare Logistic Regression and Random Forest, with and without SMOTE oversampling, using metrics suited to rare events. There is no single "best" model; the right choice depends on the business's cost of a missed fraud versus a false alarm:
- **Logistic Regression + SMOTE caught all 30 test frauds**, at the cost of 36 false alarms (1.8% of legitimate transactions).
- **Random Forest + SMOTE raised zero false alarms** but missed 8 frauds.

| | |
|---|---|
| **Dataset** | Credit card fraud: 10,000 transactions × 10 columns, 151 frauds |
| **Techniques** | Imbalance-aware EDA, feature engineering, stratified split, SMOTE, Logistic Regression, Random Forest, precision/recall/F1/AUC, confusion matrices |
| **Tools** | Python, pandas, NumPy, scikit-learn, imbalanced-learn, matplotlib, seaborn |
| **Headline result** | LR + SMOTE: recall 1.00, AUC 0.998 · RF + SMOTE: precision 1.00, F1 0.846 |

---

## 1. Business Question

How can we flag likely fraud in real time while keeping false alarms low? Every missed fraud is a direct financial loss. Every false alarm costs analyst time and annoys a customer whose card gets blocked.

## 2. Dataset

`credit_card_fraud_10k.csv`: 10,000 transactions, 0 missing values.

| Column | Description |
|---|---|
| transaction_id | Unique ID (dropped before modelling) |
| amount | Transaction amount |
| transaction_hour | Hour of day (0–23) |
| merchant_category | Clothing / Electronics / Food / Grocery / Travel |
| foreign_transaction | 1 if the transaction is abroad |
| location_mismatch | 1 if the transaction location differs from the cardholder's usual location |
| device_trust_score | Trust score of the device used (higher = more trusted) |
| velocity_last_24h | Number of card transactions in the last 24 hours |
| cardholder_age | Age of the cardholder |
| **is_fraud** | **Target**: 151 frauds (1.51%) vs 9,849 legitimate |

> **Why accuracy is not used:** a model that labels everything "not fraud" would be **98.49% accurate and catch nothing**. This project evaluates models with **precision, recall, F1 and ROC-AUC**.

## 3. Exploratory Findings

### Fraud rate by risk factor
| Segment | Transactions | Fraud rate | vs. baseline 1.51% |
|---|---:|---:|---:|
| **Hours 00:00–03:59** | 1,652 | **7.3%** | ~5× |
| All other hours | 8,348 | 0.36% | ~0.2× |
| **Foreign transaction** | 978 | **8.4%** | ~5.5× |
| Domestic | 9,022 | 0.76% | |
| **Location mismatch** | 857 | **8.4%** | ~5.5× |
| Location match | 9,143 | 0.86% | |
| Merchant category (any) | ~2,000 each | 1.2%–2.0% | ~1× |

### Fraud vs legitimate: medians
| Feature | Fraud | Legitimate |
|---|---:|---:|
| device_trust_score | **32** | **62** |
| velocity_last_24h | 3 | 2 |
| amount | $119 | $122 |
| cardholder_age | 44 | 44 |

**Takeaways**
- **Time of day is the strongest behavioural signal.** Hours 0–3 contain 16.5% of transactions but **80% of all fraud** (121 of 151). Hourly fraud rates are 6–9% at night and below 1% during the day.
- **Device trust is the strongest numeric separator.** The median for fraudulent transactions is about half that of legitimate ones.
- **Amount and merchant category barely matter.** Fraud is about *how and where* a transaction happens, not how big it is or what is bought.

## 4. Methodology

1. **Feature engineering**
   - `is_night`: flag for hours 0–3. Raw hour treats 23:00 and 00:00 as far apart even though they are adjacent on the clock, so a flag captures the night-time window better.
   - One-hot encoded `merchant_category` and dropped `transaction_id`.
2. **Split:** stratified 80/20, so the test set has **2,000 transactions with 30 frauds**.
3. **Scaling:** `StandardScaler` fitted on the training set.
4. **Imbalance handling:** **SMOTE applied to the training set only**, so no synthetic samples leak into the test evaluation.
5. **Models:** Logistic Regression and Random Forest (with SMOTE), plus Logistic Regression without SMOTE as an ablation.
6. **Evaluation:** precision, recall, F1, ROC-AUC, confusion matrices, feature importance and coefficients.

## 5. Results

| Model | Precision | Recall | F1 | ROC-AUC | Frauds caught (of 30) | False alarms |
|---|---:|---:|---:|---:|---:|---:|
| Logistic Regression + SMOTE | 0.455 | **1.000** | 0.625 | 0.998 | **30** | 36 |
| Random Forest + SMOTE | **1.000** | 0.733 | **0.846** | **1.000** | 22 | **0** |
| Logistic Regression (no SMOTE) | 0.781 | 0.833 | 0.806 | – | 25 | 7 |

**Observations**
- **SMOTE is a trade-off, not a free improvement.** On Logistic Regression it raised recall from 0.83 to 1.00 but cut precision from 0.78 to 0.46, so **F1 actually fell** (0.81 → 0.63).
- **Both models rank transactions almost perfectly (AUC ≈ 1.0).** The difference between them comes from where each model's default 0.5 threshold sits, so tuning the threshold is the main remaining lever.
- **Top predictors in both models:** device_trust_score (low trust → fraud) and is_night, followed by velocity_last_24h, foreign_transaction and location_mismatch. This matches the EDA.
- **Small test sample:** with only 30 test frauds, **each fraud moves recall by about 3.3 points**, so the differences between models are noisy.

## 6. Recommendation: Choose by Cost

| If the business priority is… | Use | Why |
|---|---|---|
| **Catching every fraud** (high loss per fraud, analysts available) | **LR + SMOTE** | 100% recall, and 36 alerts per 2,000 transactions (1.8%) is a workload a review team can handle |
| **Never blocking good customers** (limited review capacity) | **Random Forest + SMOTE** | Zero false positives, but about 1 in 4 frauds gets through |
| **A balance of both** | Random Forest with a **lower decision threshold** | AUC ≈ 1.0 means recall can probably be raised with few extra false alarms |

As immediate rule-based safeguards, **step-up authentication (e.g. OTP) for night-time, foreign or location-mismatch transactions from low-trust devices** would target the highest-risk segments identified in the EDA.

## 7. Scalability: 1 Million Transactions per Hour

- 1M transactions per hour is about **280 per second**. Scoring a single transaction with Logistic Regression or a moderately sized Random Forest takes milliseconds.
- **Training (including SMOTE) runs offline**, and only the lightweight `predict` step runs on live traffic. That step is stateless, so it scales horizontally across instances.
- Logistic Regression is the cheapest to serve. A large Random Forest needs more memory and CPU per prediction.
- Fraud patterns drift over time, so the system needs **monitoring of precision and recall, alerts on drift, and scheduled retraining**.

## 8. Limitations

- **Probably synthetic data:** the near-perfect separation (AUC ≈ 1.0) and the very clean night/foreign patterns are unusually tidy. Expect noticeably lower performance on real transactions.
- **Small number of positive cases:** 151 frauds in total, 30 in test. Stratified cross-validation would give more reliable estimates.
- **Default thresholds and hyperparameters** were used. Precision-recall curve analysis and threshold tuning are the natural next steps.
- No cost-weighted evaluation yet. Adding the average fraud loss and the cost of a review would allow a direct choice of the cheapest threshold.

## 9. How to Reproduce

```bash
pip install pandas numpy scikit-learn imbalanced-learn matplotlib seaborn jupyter
jupyter notebook Fraud_Detection.ipynb
```

```
DataAnalytics-L2-T3-FraudDetection/
├── Fraud_Detection.ipynb
└── credit_card_fraud_10k.csv
```

## Author

**Aarush Gupta**, Oasis Infobyte Summer Internship Program (Data Analytics Track)
