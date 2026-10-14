# Unveiling the Android App Market: Google Play Store Analysis

**Oasis Infobyte Internship (OIBSIP), Data Analytics, Level 2, Task 4**

This is an end-to-end analysis of about 9,700 Google Play apps and 37,000 user reviews. It cleans messy scraped data, then analyses category competition, ratings, size, pricing and review sentiment, and turns the results into a go-to-market brief for a developer launching a new app. In short: **launch free, avoid the most crowded categories, and compete on quality, not size**. The market is dominated by a few hits, in both installs and paid revenue.

| | |
|---|---|
| **Datasets** | Google Play Store Apps (Kaggle): 10,841 app rows × 13 columns, plus 64,295 user reviews |
| **Techniques** | Data cleaning and type conversion, de-duplication, descriptive and comparative analysis, Spearman correlation, revenue estimation, VADER sentiment analysis, interactive Plotly chart |
| **Tools** | Python, pandas, NumPy, matplotlib, seaborn, Plotly, NLTK (VADER) |
| **Clean data** | 9,659 unique apps · 37,427 reviews with text |

---

## 1. Business Question

**A developer is planning a new Android app. Which category should they target, how should they price it, and what do users reward or punish?**

## 2. Data

| File | Rows | Contents |
|---|---:|---|
| `googleplaystore.csv` | 10,841 | App, Category, Rating, Reviews, Size, Installs, Type, Price, Content Rating, Genres, Last Updated, Current Ver, Android Ver |
| `googleplaystore_user_reviews.csv` | 64,295 | App, Translated_Review, Sentiment, Sentiment_Polarity, Sentiment_Subjectivity (covers about 1,000 apps) |

## 3. Data Cleaning

| Issue found | Action | Impact |
|---|---|---|
| 1 corrupted row with shifted columns (Category = "1.9") | Dropped | −1 row |
| 483 exact duplicate rows | Dropped | −483 rows |
| 698 repeated app names (multiple scrapes) | Kept the record with the most reviews (the most recent) | −698 rows → **9,659 unique apps** |
| `Installs` stored as text ("10,000+") | Removed "+" and "," and cast to int | Installs are **lower bounds** |
| `Price` stored as text ("$4.99") | Removed "$" and cast to float | |
| `Size` stored as text ("19M", "201k", "Varies with device") | Converted to MB. "Varies with device" set to NaN (1,228 apps) | |
| `Reviews` and `Last Updated` stored as text | Cast to int and datetime | |
| `Type` missing in 1 row | Rebuilt from Price (Price > 0 → Paid) | 0 missing |
| `Rating` missing for 1,463 apps | **Left as NaN** (not imputed) | Mean imputation would invent ratings and flatten the distribution |
| 26,868 reviews with no text | Dropped before sentiment analysis | **37,427 usable reviews** |

## 4. Findings

### 4.1 Competition: where the apps are
| Most crowded | Apps | Least crowded | Apps |
|---|---:|---|---:|
| Family | 1,878 | Beauty | 53 |
| Game | 945 | Comics | 56 |
| Tools | 829 | Parenting | 60 |
| Business | 420 | Art & Design | 61 |
| Medical | 395 | Events | 64 |

The top 3 categories hold **38% of all apps**.

### 4.2 Demand: installs are dominated by a few hits
| Category | Apps | Mean installs/app | Median installs/app |
|---|---:|---:|---:|
| Communication | 315 | 35.0M | 100K |
| Video Players | 164 | 24.0M | 1M |
| Social | 239 | 23.0M | 100K |
| Photography | 281 | 16.6M | 1M |
| Game | 945 | 14.2M | 1M |

In Communication and Social, mean installs are **more than 200× the median**. That means a few giants (messaging and social-network apps) take most of the demand, and a typical new app there gets about 100K installs.

### 4.3 Ratings
- 8,196 rated apps: **mean 4.17, median 4.3**, with the interquartile range 4.0–4.5. Ratings are compressed near the top, so **a rating below 4.0 is a real negative signal**.
- **Highest-rated categories:** Events (4.44), Art & Design (4.36), Education (4.35), Books & Reference (4.34).
- **Lowest-rated categories:** Dating (3.98), Maps & Navigation, Tools and Video Players (all about 4.04).

### 4.4 Size vs installs
Spearman ρ = **0.31**, a weak positive relationship. Bigger apps are slightly more likely to have more installs (probably because richer apps are larger), but **size doesn't hold back adoption**.

### 4.5 Pricing and monetisation
| Metric | Free | Paid |
|---|---:|---:|
| Share of apps | **92.2%** (8,905) | 7.8% (754) |
| Median installs | **100,000** | 1,000 |
| Average rating | 4.17 | 4.26 |

- Paid apps are cheap: **median $2.99**, and **79% cost $5 or less**.
- Paid apps reach **about 100× fewer users** (by median installs), although their small, self-selected audiences rate them slightly higher.
- **20 "I am Rich"-style novelty apps cost about $400.** I excluded them from the revenue estimate because they would distort it.

**Estimated paid-app revenue** (Price × Installs, ≤ $100): **about $205M in total**
| Category | Est. revenue | Note |
|---|---:|---|
| Family | ~$106M | **Minecraft alone ≈ $70M** |
| Game | ~$41M | Hitman Sniper, GTA: San Andreas |
| Every other category | < $10M each | |

The **top 10 paid apps earn about 57% of all estimated paid revenue**. Paid-app revenue is a hit-driven market.

### 4.6 User sentiment (VADER on 37,427 reviews)
| VADER label | Share |
|---|---:|
| Positive | 68.2% |
| Negative | 20.1% |
| Neutral | 11.7% |

- **Validation:** VADER agrees with the dataset's own sentiment labels on **75.8%** of reviews. The largest disagreement is 2,791 reviews that the file labels negative but VADER scores positive. These are typically mixed or sarcastic reviews, where a lexicon model is weakest.

**Sentiment by category** (categories with ≥ 100 reviews; 35,929 reviews matched across 816 apps)
| Happiest users | Avg. compound | % negative | Most critical users | Avg. compound | % negative |
|---|---:|---:|---|---:|---:|
| Auto & Vehicles | 0.51 | 4.8% | Social | 0.16 | **31.3%** |
| Health & Fitness | 0.48 | 13.9% | News & Magazines | 0.16 | 29.9% |
| Parenting | 0.45 | 10.8% | Video Players | 0.19 | **31.1%** |
| Education | 0.45 | 12.0% | Entertainment | 0.23 | 26.7% |

**Video Players and Social combine high demand with the most frustrated users**, which suggests an opening for a better product.

## 5. Recommendations for a New App

1. **Pick a less crowded category, or bring a clear differentiator.** Family, Game and Tools are saturated. High-install categories (Communication, Social) are controlled by a few incumbents, and a typical app there gets about 100K installs. Mid-sized categories with satisfied users and room to grow (Health & Fitness, Education, Parenting) are a better fit for a new entrant.
2. **Launch free and monetise through in-app purchases or subscriptions.** Free apps reach about 100× more users, paid prices cluster at $1–$5, and paid revenue goes to a handful of hits. Charging upfront limits reach for little expected return.
3. **Compete on quality, not on a small download size.** Size barely relates to installs, while ratings are compressed near 4.3, so a below-4.0 rating stands out. In Video Players, Social and News, where about 30% of reviews are negative, a clearly better user experience is the opening.

## 6. Limitations

- **Installs are lower bounds** ("10,000+"), so install and revenue figures are approximate. Revenue also ignores Google's commission, refunds, in-app purchases and ads.
- **The reviews cover only about 1,000 apps** (816 after matching), skewed toward popular titles, so category sentiment may not represent the long tail.
- **Snapshot data (latest update in the file is August 2018):** the market has changed since then, and the analysis says nothing about trends over time.
- **VADER is a lexicon model.** It handles sarcasm and domain-specific slang poorly (75.8% agreement).

## 7. How to Reproduce

```bash
pip install pandas numpy matplotlib seaborn plotly nltk jupyter
jupyter notebook Google_Play_Store_Analysis.ipynb
```
The notebook downloads NLTK's `vader_lexicon` on its first run. The Plotly "Competition vs Demand" chart is interactive and does **not** render in GitHub's notebook preview, so run the notebook locally to see it.

```
DataAnalytics-L2-T4-GooglePlayStoreAnalysis/
├── Google_Play_Store_Analysis.ipynb
├── googleplaystore.csv
└── googleplaystore_user_reviews.csv
```

## Author

**Aarush Gupta**, Oasis Infobyte Summer Internship Program (Data Analytics Track)
