# Customer Review Auto-Sorter

![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?logo=jupyter&logoColor=white)
![NLP](https://img.shields.io/badge/NLP-VADER-orange)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Processing-150458?logo=pandas&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)

> A beginner-friendly NLP automation project that reads 100,000 customer reviews in chunks, classifies sentiment with VADER, tags topics (delivery, price, quality, service), and raises real-time alerts for angry reviews.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [How It Works](#how-it-works)
- [Tech Stack](#tech-stack)
- [Installation](#installation)
- [Usage](#usage)
- [Notebook Walkthrough](#notebook-walkthrough)
- [Sample Output](#sample-output)
- [Make It Your Own](#make-it-your-own)
- [Project Structure](#project-structure)
- [Future Improvements](#future-improvements)
- [License](#license)

---

## Overview

A shop gets thousands of customer reviews and nobody can read them all. This project builds a small program that reads them automatically and:

1. Creates a large dataset of **100,000 sample reviews**.
2. Reads them **in small pieces** instead of loading everything at once.
3. Decides whether each review is **happy, neutral or angry** (sentiment analysis).
4. Decides what each review is **about**: delivery, price, quality or service (keyword categorizing).
5. Watches **new reviews arriving live** and raises an alarm for angry ones.

Angry reviews are saved to `urgent_reviews.csv`, a ready-made to-do list for the support team. No human reads anything, yet the right people get the angry reviews.

## Features

- **Large dataset generation:** 100,000 sample reviews saved to `reviews.csv`.
- **Chunked processing:** `pandas` reads 20,000 rows at a time, so a huge file never has to fit in memory.
- **Sentiment analysis:** VADER gives each review a score from -1 (very angry) to +1 (very happy). No training needed.
- **Topic categorization:** simple keyword matching for delivery, price, quality and service.
- **Visualization:** bar chart of reviews by topic and mood.
- **Real-time alerts:** simulated live feed that prints alarms and exports angry reviews to `urgent_reviews.csv`.

## How It Works

| Step | What the program does | Concept |
|------|----------------------|---------|
| 1 | Creates 100,000 sample reviews | Our "large dataset" |
| 2 | Reads them in small pieces | Memory-efficient processing |
| 3 | Decides if each review is happy or angry | Sentiment analysis (NLP) |
| 4 | Decides what it is about | Keyword categorizing (NLP) |
| 5 | Watches new reviews and raises an alarm for angry ones | Automation and real-time |

```
reviews.csv  -->  read in pieces  -->  mood  -->  topic  -->  chart
                                                     \
new live reviews  ------------------>  mood + topic -->  ALARM + urgent_reviews.csv
```

## Tech Stack

- **Python 3.9+**
- **pandas:** tables and chunked CSV reading
- **matplotlib:** charts
- **vaderSentiment:** ready-made "mood detector" for text
- **Jupyter Notebook:** to run the project step by step

## Installation

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/customer-review-auto-sorter.git
cd customer-review-auto-sorter

# 2. Install dependencies (once)
pip install pandas matplotlib vaderSentiment

# 3. Install Jupyter if you do not have it
pip install notebook
```

## Usage

1. Start Jupyter:
   ```bash
   jupyter notebook review_auto_sorter.ipynb
   ```
2. Run the cells **from top to bottom** (`Shift + Enter`).
3. The notebook will create two files in the same folder:
   - `reviews.csv`: the 100,000 sample reviews
   - `urgent_reviews.csv`: the angry reviews found by the live-feed simulation

**Using your own data?** Skip Step 1 and put your file name in Step 2. The CSV only needs a column called `review`.

## Notebook Walkthrough

### Step 0: Load the tools

```python
import random, time
import pandas as pd
import matplotlib.pyplot as plt
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer

reader = SentimentIntensityAnalyzer()   # a ready-made "mood detector" for text
random.seed(1)                          # makes the random data the same every run
```

`pandas` handles tables, `matplotlib` draws charts, and VADER scores text from -1 to +1. `random.seed(1)` keeps the random data identical on every run.

### Step 1: Make a big pile of reviews

```python
def make_review():
    topic = random.choice(list(topics))
    mood = random.choice([good, bad])
    return random.choice(topics[topic]).format(random.choice(mood))

pd.DataFrame({"review": [make_review() for _ in range(100_000)]}).to_csv("reviews.csv", index=False)
```

`make_review()` picks a random topic and a random good or bad word, then builds a sentence such as *"Shipping was awful"*. 100,000 of them are saved to `reviews.csv`.

### Steps 2 and 3: Read in small pieces and score the mood

```python
def mood(text):
    score = reader.polarity_scores(text)["compound"]   # -1 to +1
    if score >= 0.3:
        return "happy"
    if score <= -0.3:
        return "angry"
    return "neutral"

parts = []
for piece in pd.read_csv("reviews.csv", chunksize=20_000):   # 20,000 rows at a time
    piece["mood"] = piece["review"].map(mood)
    parts.append(piece)

data = pd.concat(parts)
```

- `polarity_scores(text)["compound"]` returns the mood score from -1 to +1.
- A score of **0.3 or above** is happy, **-0.3 or below** is angry, and anything in between is neutral.
- `chunksize=20_000` hands pandas the file in pieces, so it never has to fit in memory at once.
- `.map(mood)` labels every review, and `pd.concat(parts)` glues the pieces back together.

### Step 4: Find out what each review is about

```python
keywords = {
    "delivery": ["delivery", "shipping"],
    "price":    ["price", "cost"],
    "quality":  ["quality", "build"],
    "service":  ["service"],
}

def topic_of(text):
    text = text.lower()
    for topic, words in keywords.items():
        if any(w in text for w in words):
            return topic
    return "other"

data["topic"] = data["review"].map(topic_of)
table = pd.crosstab(data["topic"], data["mood"])
```

The first topic whose keyword appears wins, and no match gives `"other"`. `pd.crosstab` counts reviews for each topic and mood combination, and the bar chart shows *where* customers are unhappy.

### Step 5: Real-time automation, watch new reviews and raise alarms

```python
def new_reviews(n=5):
    # PRETEND live feed. In real life, replace this with an API, database or file check.
    return [make_review() for _ in range(n)]

urgent = []

for round_number in range(1, 6):                 # 5 checks (use "while True:" to run forever)
    print(f"--- Check {round_number} ---")
    for text in new_reviews():
        feeling = mood(text)
        if feeling == "angry":
            urgent.append({"review": text, "topic": topic_of(text)})
            print(f"  ALARM [{topic_of(text)}]: {text}")
    time.sleep(1)                                 # wait 1 second before checking again

pd.DataFrame(urgent).to_csv("urgent_reviews.csv", index=False)
```

The `for` loop is the "watcher": it checks, handles new reviews with the same `mood()` and `topic_of()` helpers, waits a second, and checks again. Angry reviews trigger an alarm and are saved to `urgent_reviews.csv`.

## Sample Output

**Chunked processing**

```
Processed 20,000 reviews
Processed 20,000 reviews
Processed 20,000 reviews
Processed 20,000 reviews
Processed 20,000 reviews
```

**Topic vs mood counts**

| Topic | Angry | Happy |
|-------|------:|------:|
| delivery | 12,434 | 12,371 |
| price | 12,355 | 12,647 |
| quality | 12,515 | 12,609 |
| service | 12,522 | 12,547 |

> The counts are nearly balanced because the sample data is randomly generated. With real reviews, the imbalance shows where customers are unhappy.

**Live alerts**

```
--- Check 1 ---
  ALARM [price]: The price is disappointing
  ALARM [delivery]: Shipping was very poor
  ALARM [delivery]: Shipping was horrible
--- Check 2 ---
  ALARM [service]: The service team was very poor
  ...
Saved 13 urgent reviews to urgent_reviews.csv
```

> Add a screenshot of your chart here: `![Reviews by topic and mood](images/chart.png)`

## Make It Your Own

| Want to... | Change this |
|------------|-------------|
| Use real reviews | Put your CSV in Step 2 (needs a `review` column) |
| Use a real live feed | Rewrite `new_reviews()` to call an API (e.g. `requests.get(...)`) |
| Get alerts in Slack or email | Replace the `print("ALARM...")` line with a webhook or email call |
| Run forever | Change `for round_number in range(1, 6):` to `while True:` |
| Add more topics | Add lines to the `keywords` dictionary |
| Understand harder text (sarcasm, other languages) | Replace VADER with a `transformers` sentiment model |

## Project Structure

```
customer-review-auto-sorter/
├── review_auto_sorter.ipynb   # the full project, step by step
├── reviews.csv                # generated: 100,000 sample reviews
├── urgent_reviews.csv         # generated: angry reviews from the live feed
├── requirements.txt           # pandas, matplotlib, vaderSentiment
├── images/
│   └── chart.png              # topic vs mood chart (optional)
└── README.md
```

## Future Improvements

- Connect to a real review source (API or database)
- Send alerts through Slack, email or Microsoft Teams
- Replace keyword matching with a trained topic classifier
- Use a transformer-based sentiment model for sarcasm and multilingual reviews
- Build a live dashboard (for example with Streamlit)

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

---

If you found this project useful, consider giving it a star.
