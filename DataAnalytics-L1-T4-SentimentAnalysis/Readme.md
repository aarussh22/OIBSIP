# Sentiment Analysis: US Airline Tweets

**Oasis Infobyte Internship (OIBSIP), Data Analytics, Level 1, Task 4**

I built an NLP pipeline that sorts customer tweets about US airlines into **negative, neutral or positive**. The best model (TF-IDF + Logistic Regression) reaches **79.4% accuracy and 0.72 macro-F1**. That beats the 63% majority-class baseline by 16 points, and the model catches **94% of negative tweets**, which makes it useful for triaging complaints.

| | |
|---|---|
| **Dataset** | Twitter US Airline Sentiment (Kaggle): 14,640 tweets, 14,427 after de-duplication |
| **Techniques** | Text cleaning, lemmatisation, TF-IDF (uni- and bi-grams), Multinomial Naive Bayes, Logistic Regression, error analysis |
| **Tools** | Python, pandas, NumPy, NLTK, scikit-learn, matplotlib, seaborn, WordCloud |
| **Headline result** | Logistic Regression: 79.4% accuracy, 0.72 macro-F1, 0.94 negative recall |

---

## 1. Business Question

Airlines receive thousands of public tweets every day. **Can we automatically flag the unhappy ones so customer-service teams reply to them first, and track how sentiment changes over time?**

## 2. Dataset

- `Tweets.csv`: 14,640 tweets mentioning six US airlines, each hand-labelled `negative`, `neutral` or `positive`.
- Only two columns were used: `text` and `airline_sentiment`.
- **213 duplicate tweets were removed before the train/test split**, so that no tweet could appear in both sets. That leaves 14,427 tweets.

**Class balance (raw data)**

| Sentiment | Tweets | Share |
|---|---:|---:|
| Negative | 9,080 | 62.7% |
| Neutral | 3,057 | 20.9% |
| Positive | 2,290 | 15.6% |

> The data is heavily skewed toward negative. A model that always answers "negative" would already score about 63% accuracy, so that is the **baseline** to beat. Because of this skew I report **per-class recall and macro-F1** and not accuracy alone.

## 3. Methodology

1. **Text preprocessing:** lowercasing; removal of URLs, @mentions, HTML entities, `#`, punctuation, numbers and emojis; tokenisation; stopword removal; WordNet lemmatisation.
   - I kept the negation words *no*, *nor* and *not*. Without them "not good" would become "good", and these words turned out to be among the strongest negative signals.
2. **Split:** stratified 80/20, giving 11,541 train and 2,886 test tweets.
3. **Features:** a TF-IDF vectoriser **fitted on the training set only** (to avoid leakage), with the top 5,000 uni- and bi-grams.
4. **Models:** Multinomial Naive Bayes and Logistic Regression (`max_iter=1000`).
5. **Evaluation:** accuracy, per-class precision/recall/F1, confusion matrices, a review of misclassified tweets, and the most influential words per class.

## 4. Results

| Model | Accuracy | Negative recall | Neutral recall | Positive recall | Macro F1 |
|---|---:|---:|---:|---:|---:|
| *Baseline (always "negative")* | *62.9%* | *1.00* | *0.00* | *0.00* | *n/a* |
| Naive Bayes | 74.2% | 0.98 | 0.26 | 0.44 | 0.60 |
| **Logistic Regression** | **79.4%** | 0.94 | **0.49** | **0.63** | **0.72** |

**Per-class detail (Logistic Regression, test set n = 2,886)**

| Class | Precision | Recall | F1 | Support |
|---|---:|---:|---:|---:|
| Negative | 0.82 | 0.94 | 0.87 | 1,816 |
| Neutral | 0.67 | 0.49 | 0.57 | 612 |
| Positive | 0.82 | 0.63 | 0.71 | 458 |

**Reading the confusion matrices:** Naive Bayes mostly predicts "negative". It moved **431 of 612** neutral tweets and **227 of 458** positive tweets into the negative class. Logistic Regression roughly halved those errors, to **269** and **115**.

## 5. Key Insights

- **Complaints are about operations.** The top Logistic Regression features for *negative* are **bag, delayed, hr, hold, worst, hour, no, not**. These point to baggage handling, delays and phone hold times as the main pain points.
- **Praise is generic.** Positive tweets are driven by **thanks, thank, great, awesome, love, amazing, best, thx**. These are polite words that carry little information about *what* went well.
- **Neutral tweets are mostly requests.** Words like **hi, dm, question, possible, chance** suggest service questions and not opinions.
- **Word clouds need care.** All three classes are dominated by "flight". The words that actually separate the classes are smaller ones, such as *cancelled* and *delayed* versus *thank* and *great*.
- **Neutral is the hardest class** for both models, followed by positive. Both are minority classes with less distinctive vocabulary.

## 6. Error Analysis

Logistic Regression misclassified **595 of 2,886** test tweets (20.6%). The errors follow clear patterns:

| Pattern | Example | Actual → Predicted |
|---|---|---|
| A single trigger word with no context | "all set. Figured it out at the **baggage** office!" | neutral → negative |
| A change of tone partway through | "I forgot that Intl flights out of LAX don't go from Intl Terminal! Easiest re-check in ever! woo!" | positive → negative |
| Factual tweets left almost empty after cleaning | "877 from amsterdam to ewr, 02.27.2015, 737-300." | neutral → negative |
| Excitement with no sentiment words | "oh my god LAUREN OH MY GOD OH MY GOD" | positive → negative |

The common cause is that **a bag-of-words model with a majority-class bias falls back to "negative" when it finds no signal.**

## 7. Recommendations

1. **Use the model for complaint triage, not final decisions.** With 94% negative recall, it can route likely complaints to a priority queue, and agents confirm before replying.
2. **Track the negative-tweet share by airline and by week** as a lightweight customer-experience KPI.
3. **Prioritise baggage and delay communication.** These topics dominate the negative vocabulary, so proactive updates there would likely reduce complaint volume the most.

## 8. Limitations and Next Steps

- **Class imbalance:** try `class_weight='balanced'` or threshold tuning to lift neutral and positive recall.
- **No context awareness:** sarcasm and changes of tone need sequence models (e.g. a fine-tuned transformer such as DistilBERT).
- **Lost signal:** removing numbers and emojis throws away useful cues (😡, 🙏). Mapping emojis to tokens could help.
- **No hyperparameter search:** TF-IDF size, n-gram range and regularisation strength were not tuned. All tweets come from a single week (16–24 February 2015), so the vocabulary reflects that week's events.
- **Not yet explored:** the dataset also includes `airline` and `negativereason` columns. Splitting the results by airline and complaint reason would turn the classifier into a direct customer-experience benchmark.

## 9. How to Reproduce

```bash
pip install pandas numpy nltk scikit-learn matplotlib seaborn wordcloud jupyter
jupyter notebook Sentiment_Analysis.ipynb
```
The notebook downloads the NLTK resources (`stopwords`, `wordnet`, `omw-1.4`) on its first run.

```
DataAnalytics-L1-T4-SentimentAnalysis/
├── Sentiment_Analysis.ipynb
└── Tweets.csv
```

## Author

**Aarush Gupta**, Oasis Infobyte Summer Internship Program (Data Analytics Track)
