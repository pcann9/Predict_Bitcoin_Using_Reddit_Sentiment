### Is it possible to use sentiment data to predict the movement of the price of bitcoin?

Bitcoin is a cryptocurrency that completely exists on the internet. Even though it is intangible, it holds an incredible amount of value. That begs the question: Where does this value come from? The answer is that it comes from the collective belief that it is a legitimate medium of exchange. While that was the original intent, since there is a finite amount of bitcoin and because of the explosion in popularity, it has evolved from a currency to that of a speculative commodity. Now the value of bitcoin, in addition to being a legitimate medium of exchange, comes from the belief that it will appreciate in value. In conclusion, the value of bitcoin comes from discourse.

Lucky for us, we have the internet (US military funding that actually went to good use). Since bitcoin is based on the internet and most
users of bitcoin use the internet, discourse around bitcoin also occurs on the internet. While there are various bitcoin/cryptocurrency communities around the internet, the largest by far are on Reddit. In addition to housing the largest bitcoin communities, Reddit is a good representative sample of the internet commuity as a whole. This means that it contains bitcoin discourse among various communities which we could possibly then compare/contrast or analyze any way that we want to. Also Reddit was founded in 2005 while bitcoin was created in 2009 so Reddit has existed since bitcoin's inception.


So to answer our question, we will use reddit comment (or submissions if you want to) sentiment around bitcoin to try and predict the price movement of bitcoin. Our sources of data will be: 

- [bitcoin data (like price, hashrate, transactions, etc.)](https://www.blockchain.com/charts)
- [reddit comment data via BigQuery (thanks to /u/Stuck_in_the_Matrix who made this possible)](https://console.cloud.google.com)

For programs, I will use a combination of
- R (for parts 2 & 3)
- Python (for part 4)

