# Introduction
In this tutorial, I am going to predict the price movement of bitcoin by doing a sentiment analysis of reddit comments that contain the word 'bitcoin' from January 1, 2009 to September 31, 2018. Sections 2 & 3 use R, while sections 4-7 use Python. 

## 1. get_reddit_comments
In order to get the Reddit comments, I use the Google Cloud Platform in order to retrieve data from a publicly availabe datasets of all reddit comments which are located [here](https://bigquery.cloud.google.com/dataset/fh-bigquery:reddit_comments). In this section, I outline how to do so, and the steps that you have to take in order to port the data to your computer for analysis. 

## 2. clean_reddit_comments
In this section, I use R in order to clean the comment data. I also use it to filter out bots using a list that I created, which is available [here](https://github.com/pcann9/Predict_Bitcoin_Using_Reddit_Sentiment/blob/master/resources/reddit_bot_names.csv). I also filter out non-english subreddits from a list that I created, which is available [here](https://github.com/pcann9/Predict_Bitcoin_Using_Reddit_Sentiment/blob/master/resources/nonenglish_subreddits.csv). 

## 3. modify_vader_lexicon
In order to derrive sentiment from the comment data, I use VADER(alence Aware Dictionary and sEntiment Reasoner) which is a "lexicon and rule-based sentiment analysis tool that is specifically attuned to sentiments expressed in social media" that is available in Python). The github repository for VADER is located [here](https://github.com/cjhutto/vaderSentiment). In this section, I modify VADER's lexicon using R by adding domain specific sentiment terms that are relevant to my analysis. I used the Loughran-McDonald financial sentiment corpus which is available [here](https://sraf.nd.edu/textual-analysis/resources/)

## 4. language_filtering
In addition to filtering out non-english subreddits (as I did in section 2), I also use langdetect, a port of Google's language detection library in python. Its github repository is [here](https://github.com/Mimino666/langdetect). I use this, in conjunction with parallel processing, to filter out non-english comments. 

## 5. sentiment_analysis
In this section, I use VADER and its modified lexicon (which I did in section 3) in order to classify the sentiment of the comment data, in conjuntion with parallel processing. 

## 6. format_data_for_analysis
In this section, I group by date all the sentiment data that I have collected and merged it with bitcoin price data, which I downloaded from yahoo finance which you can find [here](https://finance.yahoo.com/quote/BTC-USD/). 

## 7. stationary_transformation
Here, I transform my time series data so that it is stationary. I include a guideas well as examples of how to do so. I made functions that asses the stationarity of your data as well as a resource telling you how to interpret the results and what you should do in response to those results. 

## 8. var_model
The model that I chose to forecast my data was vector autoregression, which are used for multivariate time series. I compared the results from my fitted model to test data as well as against a persistent model. 
