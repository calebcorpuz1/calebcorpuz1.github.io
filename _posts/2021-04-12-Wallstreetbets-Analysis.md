---
title: "Wallstreetbets Subreddit Analysis"
date: 2020-04-12
tags: [data science, text analysis, sentiment analysis]
header:
  image: 
excerpt: "Analyzing the relationship between wallstreetbets and historical stock data."
mathjax: "true"
---
Use this as a placeholder for now. Do an actual project summary rather than just post a link.

Repo for this project can be found [here](https://github.com/calebcorpuz1/Wallstreetbets-Analysis)

## Intro and Background
In late January 2021, GameStop, who’s stock symbol is GME, soared. This was due to a short squeeze on the stock. A short squeeze is when a stock jumps sharply higher, which forces traders who shorted the stock (bet against it) to buy the stock in order to stall or cut their losses. Forcing those who bet against the stock to buy it accelerates the stock’s increase in price, which is what we saw with GameStop (Li, 2021). 

The interesting thing about this short squeeze is that it was spurred on by amateur traders. More specifically, the subreddit wallstreetbets was the main culprit. Wallstreetbets is a subreddit community that discusses trading ideas and strategies. At the end of January, the community put a lot of money into GME, which began the short squeeze. Since then, the community has grown to over 9 million members and has become an influential source of information for many amateur traders. The community has been supported by large figures such as Elon Musk, who tweeted a link to the subreddit. Members in wallstreetbets recognized that there were funds that took large short positions on the stock, prompting the discussion for the short squeeze. Since then, we have seen some funds lose large amounts of money. One of the most notable of these funds is Melvin Capital, which lost 53% in January (Lee, 2021). There has been some controversy surrounding this whole thing. Trading platforms such as Robinhood and other brokerages started to restrict the trading on GameStop which has brought up many legal questions and infuriated the community. But this hasn’t deterred the community as it continues to grow. 
	
There is no question as to what caused the change in GameStop’s stock price, but how has the community reacted since the initial surge in price? The goal of this project is to be able to identify statistical relationships between the reddit posts and the stock. 

## Data

For the reddit posts, I downloaded a dataset from kaggle that scraped all of the posts from wallstreetbets starting from the end of January 2021. I then filtered the posts to only include posts that contain some form of the word GameStop or the stock symbol GME. To retrieve the historical stock data, I used the module yfinance in Python, which can be used to download historical market data from Yahoo! Finance. 

### Data Prep

	import re
	import string
	import numpy as np
	import pandas as pd
	import matplotlib.pyplot as plt
	from re import search
	from datetime import datetime
	from nltk.corpus import stopwords
	from nltk.tokenize import word_tokenize
	from nltk.sentiment import SentimentIntensityAnalyzer 
	from sklearn.model_selection import train_test_split
	
	df = pd.read_csv('/Users/ccorpuz/Desktop/DSC 680/Data/reddit_wsb.csv')

	def search_GME(data):		# Creates and indicator if some form of gamestop is present
	    if search("gme", data):
		x = 1
	    elif search("gamestop", data):
		x = 1
	    elif search("game stop", data):
		x = 1
	    else:
		x = 0
	    return x

	df['title'] = df['title'].astype(str) 
	df['title'] = df['title'].apply(lambda x: x.lower())

	df['title_gme_ind'] = df['title'].apply(lambda x: search_GME(x)) # search the title for a form of GME

	df['body'] = df['body'].astype(str)
	df['body'] = df['body'].apply(lambda x: x.lower())

	df['body_gme_ind'] = df['body'].apply(lambda x: search_GME(x)) # search the body for a form of GME

	df['all_gme_ind'] = df['title_gme_ind'] + df['body_gme_ind'] # total the indicator columns 

	# create a dataframe that has all the posts containing some form of gamestop in either the title or body
	df_gme = df[df['all_gme_ind'] > 0]
	
	# functions to clean text
	stop_words = set(stopwords.words('english')) 

	def remove_emoji(string):
	    emoji_pattern = re.compile("["
				   u"\U0001F600-\U0001F64F"  # emoticons
				   u"\U0001F300-\U0001F5FF"  # symbols & pictographs
				   u"\U0001F680-\U0001F6FF"  # transport & map symbols
				   u"\U0001F1E0-\U0001F1FF"  # flags (iOS)
				   u"\U00002702-\U000027B0"
				   u"\U000024C2-\U0001F251"
				   "]+", flags=re.UNICODE)
	    return emoji_pattern.sub(r'', string)

	def remove_stopword(data):
	    new_data = []
	    for word in data:
		if word not in stop_words:
		    new_data.append(word)
		else:
		    continue
	    return new_data

	def retrieve_date(data): # pulls only the year, month, and day from the timestamp
	    x = data[0:10]
	    return x


## Methods

The first thing I wanted to look at is the relationship between the posts and gamestop stock. To look at this relationship I decided to perform a sentiment analysis on the posts and compare this to the stock price. Sentiment analysis is a text classification process that is used to understand the underlying sentiment or emotion of a given text. 

To perform the sentiment analysis on the reddit posts, I chose to use the VADER approach rather than other approaches such as Textblob. My reasoning for this is that reddit is considered social media, and VADER is a sentiment analysis tool that is specifically attuned to sentiments expressed in social media (pythonprogramming.net). VADER, which stands for Valence Aware Dictionary for Sentiment Reasoning, takes a string and returns 4 categories of scores: positive, neutral, negative, and compound. I chose to use the compound, which is the normalized scores of the other three. I then created a daily aggregated compound score by taking the average compound score for each day. After retrieving the daily sentiment score I looked to see if there was any correlation between the sentiment and stock price. More specifically, stock price, stock price change, as well as lagged values for both of these. I chose to also look at lagged values because someone might read something on the subreddit but either not have the time to make the trade or just decide to make the trade the next day. It is important to remember that many of the members on wallstreetbets are amateur traders. While some have the ability to, not all of them are sitting at their desks day trading everyday. It also takes time for an idea to gain traction in a community. This is why I chose to look at lagged values. 

## Conclusion 

After running the correlations, I found no strong correlation between the daily average sentiment and the close price along with its lagged values. I also ran the correlation between the difference in sentiment each day and the day to day price change. This also showed no strong correlation. Although still weak, the strongest correlation was found between the daily average sentiment and the price change with a three period lag, which had a correlation coefficient of 0.488.
	
Another question that I wanted to answer was whether or not the volume of trades was related to the number of posts in the subreddit. I ran the correlation between the number of posts and the volume along with lagged values for volume. Again, the 3 period lag showed the strongest correlation but it was still weak with a value 0.33.

One interesting thing I noticed is that when plotting the daily average sentiment and price change, the biggest drop in sentiment saw the biggest drop in stock price.    

Although I haven't found strong correlation between the variables discussed, the huge surge of people using wallstreetbets makes me believe that this is something that could be valuable and should be monitored as time passes with more data being available.  

## Code


