# 📈 Autonomous Stock Trading System

> An end-to-end pipeline combining **NLP sentiment analysis**, **LSTM price forecasting**, and **Deep Reinforcement Learning** to train an autonomous trading agent - validated on Tesla (TSLA) stock data.

---

## 🗺️ System Overview

The pipeline runs in four stages across five Jupyter notebooks:

```
🗞️  Raw Data (tweets + OHLCV prices)
         │
         ▼
🧠  [1] Financial_Sentiment_Extraction_System.ipynb
         Extracts per-day sentiment scores using FinBERT
         │
         ▼
🔮  [2] predicting_close_LSTM.ipynb
         Trains a 3-layer LSTM to predict next-day closing prices
         │
         ▼
🔧  [3] Finalising_predicting_close_LSTM.ipynb
         Filters dates and assembles final RL training datasets
         │
         ▼
🤖  [4a] gym_anytrading_with_technical_indicators_and_sentiment.ipynb
         Trains the A2C trading agent with the full feature set

🔬  [4b] Abelation Test.ipynb
         Repeats training across feature subsets to isolate which
         inputs drive performance
```

---

## 📁 Repository Structure

```
algo-trading-system/
├── 📂 Code/
│   ├── 1. Sentiment Extraction/
│   │   └── 🧠 Financial_Sentiment_Extraction_System.ipynb
│   ├── 2. Close Price Prediction/
│   │   ├── 🔮 predicting_close_LSTM.ipynb
│   │   └── 🔧 Finalising_predicting_close_LSTM.ipynb
│   └── 3. RL Logic/
│       ├── 🤖 gym_anytrading_with_technical_indicators_and_sentiment.ipynb
│       └── 🔬 Abelation Test.ipynb
└── 📂 Data/
    ├── Tweets data/
    │   ├── tweets.csv
    │   └── processed_tweets.csv
    └── TESLA data/
        ├── 1. Raw data/
        │   └── PAPER_TSLA_data.csv
        ├── 2. Processed data/
        │   ├── processed_PAPER_TSLA_data.csv
        │   └── processed_PAPER_TSLA_data_probabilities.csv
        ├── 3. Normalised data/
        │   ├── processed_normalised_PAPER_TSLA_data.csv
        │   └── processed_normalised_PAPER_TSLA_data_probabilities.csv
        ├── 4. LSTM data/
        │   ├── processed_normalised_PAPER_TSLA_data_LSTM.csv
        │   └── processed_normalised_PAPER_TSLA_data_probabilities_LSTM.csv
        └── 5. RL training data/
            ├── TESLA RL training data - 1 Sentiment 12.2024.csv
            └── TESLA RL training data - 3 Sentiment 12.2024.csv
```

---

## 📦 Required Input Data

### 🗞️ 1. News / tweet headlines CSV

In our research we used this [Kaggle dataset](https://www.kaggle.com/datasets/drlove2002/tesla-news-from-tweeter) of Tesla-related tweets.

Required columns:
- `Date` - publication date of the headline
- `Tweet` - headline or news text

### 💹 2. Stock price CSV

We used [Yahoo Finance](https://uk.finance.yahoo.com/) to download TSLA OHLCV data (2013–2020).

Required columns:
- `Date` - trading day
- `Open`, `High`, `Low`, `Close`, `Adj Close`, `Volume`

---

## 🚀 Usage: Step-by-Step

### 🧠 Step 1 - Extract Sentiment

**Notebook:** `Code/1. Sentiment Extraction/Financial_Sentiment_Extraction_System.ipynb`

Cleans raw tweets, runs them through **FinBERT** (a pre-trained financial BERT model), and aggregates daily sentiment scores. 
Days with no news receive a neutral score of `0`.

Configure the `ADD_ONE_SENTIMENT_COLUMN` flag to choose the output format:
- ✅ `True` → a single `Sentiment` column with values `{-1, 0, 1}`
- 🔢 `False` → three probability columns: `Positive Sentiment`, `Negative Sentiment`, `Neutral Sentiment`

In our research, the `ADD_ONE_SENTIMENT_COLUMN` flag is set to `True`.

The notebook also Z-score normalises the financial features (`Open`, `High`, `Low`, `Close`, `Adj Close`, `Volume`).

**📤 Outputs:**
- `Data/Tweets data/processed_tweets.csv`
- `Data/TESLA data/2. Processed data/processed_PAPER_TSLA_data.csv`
- `Data/TESLA data/3. Normalised data/processed_normalised_PAPER_TSLA_data.csv`

---

### 🔮 Step 2 - Predict Closing Prices

**Notebook:** `Code/2. Close Price Prediction/predicting_close_LSTM.ipynb`

Trains a **3-layer LSTM** network on 5 features (`Open`, `High`, `Low`, `Close`, `Volume`) using an 80-day sliding window.
Predictions are inverse-transformed back to the original price scale and saved in a new `Predicted_Close` column.

**⚙️ Key Hyperparameters:**

| Parameter | Value |
|---|---|
| 🪟 Window size | 80 days |
| 🏗️ LSTM layers | 3 |
| 🧩 Hidden units | 150 |
| 💧 Dropout | 60% |
| ✂️ Train/val split | 95% / 5% |
| ⚡ Optimizer | Adam (lr=0.001) |
| 🔁 Max epochs | 50 (early stopping, patience=10) |

**📤 Output:** `Data/TESLA data/4. LSTM data/processed_normalised_PAPER_TSLA_data_LSTM.csv`

---

### 🔧 Step 3 - Finalise RL Training Data

**Notebook:** `Code/2. Close Price Prediction/Finalising_predicting_close_LSTM.ipynb`

Filters the LSTM-augmented data to dates from 2014-01-01 onwards and produces two versioned CSV files - one for each sentiment format - ready for RL training.

**📤 Outputs:**
- `Data/TESLA data/5. RL training data/TESLA RL training data - 1 Sentiment MM.YYYY.csv`
- `Data/TESLA data/5. RL training data/TESLA RL training data - 3 Sentiment MM.YYYY.csv`

---

### 🤖 Step 4a - Train the RL Trading Agent

**Notebook:** `Code/3. RL Logic/gym_anytrading_with_technical_indicators_and_sentiment.ipynb`

Trains an **A2C (Advantage Actor-Critic)** agent inside a custom `gym-anytrading` environment while setting a trading `WINDOW_SIZE`.
The agent learns a long/short trading policy over ~1,258 training days (up to 2019-01-01) and is evaluated on ~252 test days.

In our research, `WINDOW_SIZE` was tested across `{10, 15, 20, 25}`.

**🏆 Performance is reported as:** Calmar Ratio · Total Reward · Total Profit

---

### 🔬 Step 4b - Ablation Study

**Notebook:** `Code/3. RL Logic/Abelation Test.ipynb`

Runs the same A2C training loop for the selected `FEATURE_STATE` to measure the contribution to final trading performance.
The `FEATURE_STATE` parameter controls which signals the agent observes:

| 🏷️ FeatureState | 📡 Signals |
|---|---|
| `BasicMode` | Close price only |
| `WithTechInd` | Close + `SMA, RSI, MOM, EMA, AROONOSC` |
| `WithPredict` | Above + `Predicted_Close` |
| `WithLag` | Above + `lag features` |
| `FullMode` | Above + `Sentiment` |

---

## 🛠️ Key Technologies

| Component | Library / Model |
|---|---|
| 🧠 Sentiment analysis | ![FinBERT](https://img.shields.io/badge/FinBERT-ProsusAI-blue) |
| 🔮 Price prediction | ![PyTorch](https://img.shields.io/badge/PyTorch-LSTM-EE4C2C?logo=pytorch&logoColor=white) |
| 🎮 RL environment | ![Gym](https://img.shields.io/badge/gym--anytrading-custom%20StocksEnv-brightgreen) |
| 🤖 RL algorithm | ![SB3](https://img.shields.io/badge/Stable--Baselines3-A2C%20MlpPolicy-9cf) |
| 📊 Technical indicators | ![finta](https://img.shields.io/badge/finta-SMA%20%7C%20RSI%20%7C%20MOM%20%7C%20EMA%20%7C%20AROON-yellow) |
| 🔢 Data / ML utilities | ![pandas](https://img.shields.io/badge/pandas%20%7C%20numpy%20%7C%20scikit--learn-data%20stack-150458?logo=pandas&logoColor=white) |
