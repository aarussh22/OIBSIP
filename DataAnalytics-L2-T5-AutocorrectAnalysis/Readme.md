# Autocomplete and Autocorrect: N-gram and Edit-Distance Models

**Oasis Infobyte Internship (OIBSIP), Data Analytics, Level 2, Task 5**

I built and evaluated two keyboard-style NLP features from scratch on a single public-domain novel:
- **Autocomplete** (next-word prediction) with bigram and trigram-with-backoff models. The true next word is in the **top 3 suggestions about 25% of the time**, with 96% coverage.
- **Autocorrect** (spelling correction). An off-the-shelf corrector (pyspellchecker) is compared with my own Levenshtein-distance corrector: **28/30 vs 23/30 misspellings fixed**, and **neither changed any correctly spelled word**.

| | |
|---|---|
| **Corpus** | *The Adventures of Sherlock Holmes* (Arthur Conan Doyle, Project Gutenberg): ~104,500 words |
| **Techniques** | Sentence/word tokenisation, n-gram language models with backoff, Levenshtein edit distance, top-k evaluation |
| **Tools** | Python, pandas, NLTK, pyspellchecker, `collections` (Counter, defaultdict), matplotlib, seaborn |
| **Headline result** | Trigram top-3 F1 = 0.250; pyspellchecker correction F1 = 0.933 |

---

## 1. Objective

Find out how far simple, explainable NLP techniques can go on autocomplete and autocorrect, measure them with proper held-out metrics, and explain the gap to production systems such as Google Keyboard (Gboard).

## 2. Corpus and Preprocessing

`Sherlock_holmes.txt` was downloaded from Project Gutenberg. The licence header and footer are removed automatically. The loader reads **every `.txt` file in the folder**, so you can enlarge the corpus by dropping in more books.

| Statistic | Value |
|---|---:|
| Characters | 562,214 |
| Approx. words | 104,506 |
| Sentences (≥ 3 tokens) | 4,610 |
| Tokens | 105,140 |
| Unique words (vocabulary) | 7,936 |

**Preprocessing:** curly quotes normalised (so contractions like *don't* survive), honorifics protected from sentence splitting (*Mr.*, *Dr.*), lowercasing, and regex tokenisation that removes punctuation and numbers. Sentences are kept separate, so n-grams never span a sentence boundary.

**When to remove stopwords.** I removed stopwords **only for the word-frequency chart**. The n-gram models and spell checkers keep them, because *the*, *of* and *and* are exactly the words people type most.

| Top words: all tokens | Top words: stopwords removed |
|---|---|
| the, and, i, to, of, a, … (function words) | **said, upon, holmes, one, would, man, …** (the voice of the book) |

## 3. Autocomplete

### Method
- **Bigram model:** predicts the next word from the previous 1 word.
- **Trigram + backoff:** predicts from the previous 2 words, and backs off to the bigram when that pair was never seen in training.
- **Split:** random 80/20 by sentence (seed 42): 3,688 train and 922 test sentences. Training produced 6,839 bigram contexts and 38,379 trigram contexts.

### Sample predictions (top 3)

| Prefix | Bigram | Trigram + backoff |
|---|---|---|
| sherlock | holmes | holmes |
| i am | sure, not, afraid | sure, not, afraid |
| my dear | fellow, young, little | fellow, young, holmes |
| what do you | have, are, will | **think, know**, not |
| in the | door, other, matter | **morning**, matter, world |
| baker street | and, a, which | **rooms**, that, it |

The trigram model gives more context-aware suggestions ("what do you" → *think/know*; "baker street" → *rooms*).

### Evaluation (19,738 next-word positions in held-out sentences)

| Model | Top-k | Coverage | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|
| Bigram | 1 | 0.96 | 0.139 | 0.133 | 0.136 |
| Bigram | 3 | 0.96 | 0.247 | 0.237 | 0.242 |
| Trigram + backoff | 1 | 0.96 | 0.148 | 0.142 | 0.145 |
| **Trigram + backoff** | **3** | 0.96 | **0.255** | **0.245** | **0.250** |

*Coverage* = share of positions where the model offered any suggestion. *Precision* = hits ÷ positions with a suggestion. *Recall* = hits ÷ all positions. A hit means the true next word appears in the top k.

**Observations**
- Moving from 1 to 3 suggestions **almost doubles the hit rate** (≈14% → ≈25%). This is why keyboards show 3 suggestions.
- The trigram model beats the bigram by **only about 1 point**. With roughly 105K tokens, the 38K trigram contexts are mostly seen only once or twice, so the model often falls back to bigram counts. More data would help the trigram more than any change to the model.

## 4. Autocorrect

### Method
- **pyspellchecker:** a frequency-ranked dictionary corrector that searches up to edit distance 2.
- **Custom Levenshtein corrector:** returns the word unchanged if it is in the corpus vocabulary. Otherwise it picks the corpus word with the smallest edit distance (at most 2) and breaks ties by corpus frequency.
- **Test set:** 30 misspellings (missing, extra, swapped and wrong letters) plus **30 correctly spelled control words**, including names like *holmes*, *watson*, *lestrade* and *adler*, to check that the correctors do no harm.

### Results

| Method | Misspellings fixed | Correct words kept | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|
| **pyspellchecker** | **28 / 30** | 30 / 30 | **0.933** | **0.933** | **0.933** |
| Custom Levenshtein | 23 / 30 | 30 / 30 | 0.793 | 0.767 | 0.780 |

*Precision* = correct fixes ÷ all words changed. *Recall* = correct fixes ÷ 30 misspellings.

### Failure analysis

| Misspelling | Intended | pyspellchecker | Custom | Why it failed |
|---|---|---|---|---|
| wich | which | with ✗ | with ✗ | Two candidates 1 edit away, and the more frequent one wins |
| windw | window | wind ✗ | window ✓ | pyspellchecker's general frequency favours *wind*. The corpus favours *window* |
| recieve | receive | ✓ | relieve ✗ | **Swapped letters cost 2 edits** in Levenshtein, so a 1-edit word wins |
| thier | their | ✓ | thief ✗ | Swap (same cause) |
| wierd | weird | ✓ | were ✗ | Swap. *were* is far more frequent in the book |
| freind | friend | ✓ | find ✗ | Swap |
| becuase | because | ✓ | became ✗ | Swap |
| tomorow | tomorrow | ✓ | unchanged ✗ | *tomorrow* never appears in the book, so it isn't in the vocabulary |

**Takeaways**
- **5 of the 7 custom-corrector failures are letter transpositions.** Switching to Damerau-Levenshtein distance (which counts a swap as 1 edit) would likely fix most of them.
- A **domain vocabulary** helps in some cases (*windw → window*) and hurts in others (*tomorow*). The best results would come from combining a general dictionary with corpus-specific frequencies.
- pyspellchecker was also much faster. The custom corrector scans the whole vocabulary for each word and took over 2 seconds for 60 words.

## 5. Gap to Production Keyboards (e.g. Gboard)

| Dimension | This project | Production systems |
|---|---|---|
| Data | One Victorian novel (~105K words) | Billions of words of modern, varied text |
| Context | Last 1–2 words | Neural language models that read the whole sentence |
| Error types | Non-words only, no context | Real-word errors (*form* vs *from*), keyboard-adjacency typo models |
| Personalisation | None | Learns the user's names, slang and emoji, and supports many languages |
| Latency | Brute-force vocabulary scan | Indexed search (e.g. SymSpell, BK-trees) running on the device |

## 6. Next Steps

- Add more Gutenberg books to cut trigram sparsity, then re-measure the trigram vs bigram gap.
- Add Kneser-Ney smoothing for better probabilities on unseen n-grams.
- Use Damerau-Levenshtein distance and a keyboard-adjacency cost in the custom corrector.
- Re-rank correction candidates by context (n-gram probability of each candidate in the sentence).

## 7. How to Reproduce

```bash
pip install pandas nltk pyspellchecker matplotlib seaborn jupyter
jupyter notebook Autocomplete_Autocorrect.ipynb
```

```
DataAnalytics-L2-T5-AutocorrectAnalysis/
├── Autocomplete_Autocorrect.ipynb
└── Sherlock_holmes.txt        # add more .txt books here to enlarge the corpus
```

## Author

**Aarush Gupta**, Oasis Infobyte Summer Internship Program (Data Analytics Track)
