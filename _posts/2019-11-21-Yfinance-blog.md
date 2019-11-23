---
layout: post
title: "Getting Historical Data from Yahoo Finance"
date:   2019-11-21
categories: NLP
author: "Darius Choi"
---

# Yahoo  Finance API  
In our project, historical data in the financial market is critical for us to formulate a forecast based on our observation. Bloomberg is an ideal tool to obtain reliable and accurate data. However, due to the current situation in Hong Kong, accessibility to the Bloomberg Terminal might be a problem. So, I decided to rely on Yahoo Finance. I planned to write a programme to request data from the Yahoo Finance API. The Yahoo Finance API is an official way to request data from the Yahoo database. Unfortunately, when I was searching for instructions online, I realised that Yahoo Finance API was shut down in 2017. Then I came across yfinance package for a substitute.  

# The Powerful yfinance Package  
yfinance package is a powerful alternative to Yahoo Finance API. Before utilising this package, you have to install yfinance in your computer with command line “pip install yfinance --upgrade --no-cache-dir”. You may refer to this webpage for more details about yfinance: https://pypi.org/project/yfinance/. The code here is straight forward. First I list out all the tickers I need. Then I set the start date and end date. What yfinance does is to download the data of multiple tickers all at once. Apart from yfinance, I also import pandas package. The package helps me to unify the number format. By using the pandas package, I can convert all the numbers into 4 decimal places, particularly important to deal with the scientific numbers displayed in the “Volume” column by default. This problem might be due to the excessively huge figures. Next, in the raw data, it contains “Open”, “High”, “Low”, “Close”, “Adj Close” and “Volume”, which refer to open price, highest day price, lowest day price, close price, adjusted close price and trading volume respectively. “Adj Close” and “Volume” are more relevant to our project at this stage, so I only store these 2 categories into a tsv file delimited by tabs. By this, I do not imply the highest or lowest day prices are unimportant. They might useful for us to conduct intraday sentimental analysis. However, at the current stage, we are trying to identify any predictable patterns between public sentiments and market movements only on a daily or even weekly basis, making intraday analysis less relevant at this point of time.
