# Autonomous Stock Trading System
Submitted by:<br>
Shartil Shartilov - 22085025<br>
Riaz Khan – 23068308<br>
Najeeb Ullah - 23061856
<hr>

## Required Data for the System
In order to use the system, we require two types of data:
1. CSV file containing news headlines regarding desired stock<br>
In our research, we used this [Kaggle dataset](https://www.kaggle.com/datasets/drlove2002/tesla-news-from-tweeter) containing news headlines regarding the TSLA stock.<br>
Each line in this file also contains a source link for the original website where given headline was published.
<br><br>
This file must contain the next columns:<br>
1.1 News headline date.<br>
1.2 News headline content.


2. CSV file containing the desired stock data<br>
In our research, we used this [Kaggle dataset](https://www.kaggle.com/datasets/varpit94/tesla-stock-data-updated-till-28jun2021) containing different TSLA stock price data for over a decade.
<br><br>
This file must contain the next columns:<br>
2.1 Trading day date.<br>
2.2 Open stock price.<br>
2.3 High stock price.<br>
2.4 Low stock price.<br>
2.5 Close stock price.<br>
2.6 Adjusted close stock price.<br>
2.7 Stock Volume.
<hr>

## System Usage
After gathering the required pieces of data, we can start using the system.<br>
Here are the steps for using the system by running the different jupyter notebooks:
1. Extract financial sentiment from news headlines CSV file<br>
Run ```Financial_Sentiment_Extraction_System.ipynb``` notebook for extracting the financial sentiment for each trading day.
The final output from this notebook should be a new column called ```Sentiment```, added to the stock data CSV file.
This new column contains the sentiment score, where sentiment score ∈ {-1, 0, 1}.
This output should be saved under the name ```processed_TSLA.csv```.<br>

2. Predicate TSLA Close Prices<br>
Run ```predicting_close_LSTM.ipynb``` notebook for predicating TSLA close prices.<br>
In this notebook, we train a LSTM model that learns the hidden patterns between TSLA close prices.<br>
Using this model, we are able to predict the closing price and save this value in a new column called ```Predicted_Close```, which is added to the ```processed_TSLA.csv``` file.<br>
This output should be saved under the name ```Processed_predicted_TSLA.csv```.<br>

3. Training Deep Reinforcement Learning Agent<br>
At this stage, we should have a stock data CSV file with additional columns ```Sentiment``` and ```Predicted_Close```.<br>
Here, we have a wide selection of notebooks stored at the [Gym Anytrading Notebooks folder](https://github.com/mega5800/algo-trading-system/tree/master/Code/Gym%20Anytrading%20Notebooks).<br>
In each notebook, we train a deep reinforcement learning agent with a different set of available data in its environment.
By doing so, we can assess what are the effective pieces of information for training an optimal agent for autonomous stock trading.
