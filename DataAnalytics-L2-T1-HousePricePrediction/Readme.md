# House Price Prediction: Linear Regression

**Oasis Infobyte Internship (OIBSIP), Data Analytics, Level 2, Task 1**

I built an interpretable linear regression model that predicts house sale prices in Ames, Iowa from 8 hand-picked features. It explains **83% of the variance in price (R² = 0.830)** with a typical error of **about $36K (RMSE)**. Its coefficients show that **location is worth as much as 1,400 sq ft of extra living space**.

| | |
|---|---|
| **Dataset** | House Prices: Advanced Regression Techniques (Kaggle, Ames Housing): 1,460 sales × 81 columns |
| **Techniques** | Feature selection, one-hot encoding, train/test split, OLS regression, residual diagnostics, Ridge and Lasso comparison |
| **Tools** | Python, pandas, NumPy, scikit-learn, matplotlib, seaborn |
| **Headline result** | R² 0.830 · RMSE $36,077 on 292 held-out homes |

---

## 1. Objective

Predict `SalePrice` from a small set of features (size, quality, age, rooms and location) and, just as importantly, **explain which factors move price and by how much**, so the result is useful to buyers, sellers and appraisers.

## 2. Dataset

`train.csv` contains 1,460 residential sales in Ames, Iowa (2006–2010) with 79 explanatory variables.

**Target: `SalePrice`**
| Statistic | Value |
|---|---:|
| Mean | $180,921 |
| Median | $163,000 |
| Min / Max | $34,900 / $755,000 |
| Skewness | 1.88 (long right tail) |
| Homes above $400K | 28 (1.9%) |

## 3. Feature Selection

| Feature | Description | Correlation with SalePrice |
|---|---|---:|
| OverallQual | Material and finish quality (1–10) | **0.79** |
| GrLivArea | Above-ground living area (sq ft) | **0.71** |
| GarageCars | Garage capacity (cars) | 0.64 |
| TotalBsmtSF | Basement area (sq ft) | 0.61 |
| FullBath | Full bathrooms | 0.56 |
| YearBuilt | Construction year | 0.52 |
| BedroomAbvGr | Bedrooms above ground | 0.17 |
| Neighborhood | 25 neighbourhoods, one-hot encoded (`drop_first`, baseline = Bloomington Heights) | n/a |

None of the selected columns had missing values. After encoding, the model uses **31 features**.

## 4. Methodology

1. Selected the features and one-hot encoded `Neighborhood`.
2. Random 80/20 split (`random_state=42`): **1,168 train / 292 test**.
3. Fitted `LinearRegression`.
4. Evaluated with MSE, RMSE and R² on the test set.
5. Diagnostics: actual-vs-predicted plot, residual plot, coefficient analysis.
6. Compared against `Ridge(alpha=1)` and `Lasso(alpha=1)`.

## 5. Results

| Model | R² (test) | RMSE (test) |
|---|---:|---:|
| **Linear Regression** | **0.8303** | **$36,077** |
| Ridge (α = 1) | 0.8307 | – |
| Lasso (α = 1) | 0.8304 | – |

- The RMSE of about $36K is **about 22% of the median price**. That is good enough for a first estimate, but not for a final valuation.
- Ridge and Lasso change R² by **less than 0.001**, so the plain model **is not overfitting**. With only 31 features and 1,168 training rows, there's little variance for regularisation to remove.

### Diagnostics
- **Actual vs predicted:** points sit close to the diagonal for **$100K–$300K** homes and spread out above **$400K**, where there are only 28 training examples.
- **Residuals:** mostly random around zero, but they **grow and turn positive at high predicted prices**. The model **systematically under-prices expensive homes**, a sign of heteroscedasticity caused by the skewed target.

## 6. What Drives Price (coefficients)

All other features are held constant in each comparison below.

| Driver | Effect on predicted price |
|---|---|
| **Neighborhood: StoneBr / NridgHt / NoRidge** | **+$71K to +$75K** vs the baseline neighbourhood |
| Neighborhood: Veenker / ClearCr / Crawfor | +$48K to +$64K |
| Neighborhood: BrDale / Blueste | −$12K to −$14K |
| OverallQual | **+$15,380 per quality point** |
| GarageCars | +$12,510 per extra car space |
| GrLivArea | +$52 per sq ft (≈ +$34K across the middle 50% range of 1,130–1,777 sq ft) |
| YearBuilt | +$263 per year newer |
| TotalBsmtSF | +$19 per sq ft |
| FullBath | −$735 per bathroom |
| BedroomAbvGr | **−$4,668 per bedroom** |

**Interpretation**
- **Location is the biggest lever.** Moving from the baseline neighbourhood to StoneBr adds about $75K, which is the same as adding about 1,400 sq ft of living area.
- **Quality beats quantity.** One step up in overall quality (+$15K) is worth about 300 sq ft.
- **Extra bedrooms carry a negative coefficient.** Once size and quality are fixed, more bedrooms means the same space is split into smaller rooms, and buyers don't pay for that. (FullBath's negative sign is small and partly reflects its overlap with GrLivArea.)

## 7. Recommendations

1. **Appraisers and pricing tools** should weight neighbourhood heavily. Comparable sales from the same neighbourhood are worth more than square-footage matching.
2. **Homeowners planning renovations** should invest in **quality and finish** (OverallQual) or **real added space**, not in converting existing space into extra bedrooms.
3. **For high-end listings above about $400K**, don't rely on this model alone, because it under-predicts them. Use a separate model or specialist comparable sales.

## 8. Limitations and Next Steps

- **Log-transform `SalePrice`** to handle the skew. This should fix the under-prediction at the top end and the funnel-shaped residuals.
- **Add more features:** overall condition, lot area, remodel year, kitchen quality. Engineered features (house age at sale, total square footage) could help too.
- **Validation:** use 5-fold cross-validation instead of a single split, and tune α for Ridge and Lasso instead of using the default α = 1.
- **Try non-linear models** (Random Forest, Gradient Boosting) as a benchmark. They give up some interpretability in exchange for accuracy.
- The data covers one US city from 2006–2010, so the coefficients don't transfer to other markets or periods.

## 9. How to Reproduce

```bash
pip install pandas numpy scikit-learn matplotlib seaborn jupyter
jupyter notebook House_Price_Prediction.ipynb
```

```
DataAnalytics-L2-T1-HousePricePrediction/
├── House_Price_Prediction.ipynb
└── train.csv
```

## Author

**Aarush Gupta**, Oasis Infobyte Summer Internship Program (Data Analytics Track)
