# Wine Quality Prediction: Classification from Chemistry

**Oasis Infobyte Internship (OIBSIP), Data Analytics, Level 2, Task 2**

I predict whether a wine is **Low, Medium or High quality** from 11 lab measurements, and compare three classifiers: Random Forest, SGD and SVC. **Random Forest is clearly the best model (72.8% accuracy, 0.72 macro-F1)**. It is also the only one that reliably identifies premium wines (High-class recall 0.61, against 0.34 for SVC and 0.04 for SGD).

| | |
|---|---|
| **Dataset** | Wine Quality (UCI / Kaggle), red and white merged: 6,497 wines × 13 columns |
| **Techniques** | EDA, class-imbalance handling through target binning, stratified split, feature scaling, ensemble/linear/kernel classifiers, feature importance |
| **Tools** | Python, pandas, scikit-learn (RandomForestClassifier, SGDClassifier, SVC, StandardScaler), matplotlib, seaborn |
| **Headline result** | Random Forest: 72.8% accuracy · 0.72 macro-F1 |

---

## 1. Objective

Can a winery estimate a wine's quality tier from routine chemical analysis, before expert tasting? Which chemical properties matter most?

## 2. Dataset

`wine_quality_merged.csv` contains Portuguese *Vinho Verde* wines: **4,898 white and 1,599 red**.

- **11 features:** fixed acidity, volatile acidity, citric acid, residual sugar, chlorides, free and total sulfur dioxide, density, pH, sulphates, alcohol
- **Target:** `quality`, an expert score from 0 to 10 (observed range 3–9)
- **Data quality:** 0 missing values and all features numeric. `type` (red/white) was not used as a feature.

### Target distribution: highly imbalanced
| Score | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|---|---:|---:|---:|---:|---:|---:|---:|
| Wines | 30 | 216 | 2,138 | 2,836 | 1,079 | 193 | **5** |

Scores 5–7 cover **93% of the wines**. With only 5 examples of score 9 and 30 of score 3, a 7-class model can't learn the rare grades.

### Binning decision
| Class | Rule | Wines | Share |
|---|---|---:|---:|
| Low | quality ≤ 5 | 2,384 | 36.7% |
| Medium | quality = 6 | 2,836 | 43.7% |
| High | quality ≥ 7 | 1,277 | 19.7% |

Three classes give a much more balanced target while keeping more detail than a good/bad split, and they map onto real price tiers (everyday, mid-range and premium).

## 3. Exploratory Findings

| Feature | Correlation with quality |
|---|---:|
| **Alcohol** | **+0.44** |
| Density | −0.31 |
| Volatile acidity | −0.27 |
| Chlorides | −0.20 |
| All others | between −0.08 and +0.09 |

- **Alcohol is the strongest single signal.** Average alcohol rises from **9.8% at score 5 to 11.4% at score 7 and 12.2% at score 9**.
- **Alcohol and density are strongly correlated with each other (−0.69)**, so they carry overlapping information. Linear models will have trouble separating their effects.
- No single feature separates the classes cleanly, which suggests that **non-linear models** should do better.

## 4. Methodology

1. Stratified 80/20 split (`random_state=42`): **5,197 train / 1,300 test**.
2. Fitted `StandardScaler` on the training data only. It is required for SGD and SVC and not needed for Random Forest.
3. Trained `RandomForestClassifier`, `SGDClassifier` and `SVC`, all with default hyperparameters.
4. Evaluated each model with accuracy, per-class precision/recall/F1 and a confusion matrix, plus Random Forest feature importances.

## 5. Results

| Model | Accuracy | Low recall | Medium recall | **High recall** | Macro F1 |
|---|---:|---:|---:|---:|---:|
| **Random Forest** | **72.8%** | 0.75 | 0.76 | **0.61** | **0.72** |
| SVC (RBF) | 62.9% | 0.70 | 0.70 | 0.34 | 0.59 |
| SGD (linear) | 55.1% | 0.60 | 0.74 | 0.04 | 0.43 |
| *Majority baseline (always "Medium")* | *43.6%* | – | – | *0.00* | – |

Test-set support: Low 477, Medium 567, High 256.

**Observations**
- **Random Forest wins every class**, with F1 of 0.77 for Low, 0.71 for Medium and 0.68 for High.
- **SGD's 55% accuracy hides a near-total failure on premium wines.** It found only 4% of High wines and predicts almost entirely Low or Medium. This is a good example of why accuracy alone is misleading on imbalanced data.
- **High is the hardest class for every model** because it has the fewest training examples.
- **Feature importance (Random Forest):** alcohol, density and volatile acidity rank highest, which matches the correlation analysis.

## 6. Recommendations

1. **Use Random Forest** for quality screening. It is the most accurate model, the most balanced across classes, and its feature importances can be explained to winemakers.
2. **Monitor alcohol and density during production** as early indicators of a batch's quality tier.
3. **Control volatile acidity** (the vinegar-like fault) during fermentation and storage. It is the strongest *negative* chemical driver that the producer controls.
4. **Don't yet rely on the model to certify premium wines.** With 61% High recall, about 4 in 10 premium wines would be missed. Collect more high-scoring samples, or use class weighting, before using it for that decision.

## 7. Limitations and Next Steps

- **Duplicate rows (important):** the merged file contains **1,177 exact duplicate rows (18%)**. A random split can put identical wines in both train and test, which **probably inflates the test scores**, especially for Random Forest. Next step: de-duplicate before splitting and re-evaluate.
- **No tuning or cross-validation:** all three models use default settings, so SVC and SGD in particular are under-tuned. Next steps: grid search, `class_weight='balanced'`, and 5-fold stratified CV for more stable estimates.
- **Wine type ignored:** red and white wines have different chemistry. Adding `type` as a feature, or training separate models, may improve results.
- **Subjective target:** quality is the median score of human tasters, so some label noise is unavoidable.

## 8. How to Reproduce

```bash
pip install pandas scikit-learn matplotlib seaborn jupyter
jupyter notebook Wine_Quality_Prediction.ipynb
```
> Note: cell 9 of the notebook contains markdown text in a code cell, which raises a `SyntaxError` when the notebook is run top to bottom. Change that cell's type to Markdown before using "Run All".

```
DataAnalytics-L2-T2-WineQualityPrediction/
├── Wine_Quality_Prediction.ipynb
└── wine_quality_merged.csv
```

## Author

**Aarush Gupta**, Oasis Infobyte Summer Internship Program (Data Analytics Track)
