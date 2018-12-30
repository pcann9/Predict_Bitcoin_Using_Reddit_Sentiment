# Modify VADER Lexicon in R
Since we now have our cleaned reddit comments, we need to figure out a way to derive sentiment from those comments and then figure out a way to then quantify that sentiment. Before we do so, the method that we choose is going to have to solve three problems: 
1. Our data is not prelabelled and we don't have a metric to try and predict (like stars on a review)
2. We don't have predefined sentiment dictionaries/lexicon of positive/negative words
3. We need domain specific sentiment words around bitcoin as well as around finance. Financial sentiment is *very* different than normal sentiment. 

To best meet all of these needs, I chose [VADER](https://github.com/cjhutto/vaderSentiment) (Valence Aware Dictionary and sEntiment Reasoner) which is a "lexicon and rule-based sentiment analysis tool that is specifically attuned to sentiments expressed in social media" that is available in Python.

### 1. Download VADER
You can download VADER from its github repository [here](https://github.com/cjhutto/vaderSentiment), or by conda/pip.

### 2. Get VADER's Lexicon Text file
While VADER does have a comprehensive lexicon, we may want to add sentiment words that are specific to our domain. In order to do so, go to the file location where python is downloaded to and go `Python_File_Location>Lib>site-packages>vaderSentiment`. This is the location of the source files of this package. You will notice that there is a text file called "vader_lexicon.txt". This is the location of the VADER lexicon with all of the sentiment words as well as their respective weight. **COPY** and paste this file to your R working_directory. You can also download the [vader lexicon text file here](https://github.com/pcann9/reddit-bitcoin-prediction/blob/master/resources/vader_lexicon(real).txt). 

### 3. Get Domain Specific Lexicon
While you could make sentiment dictionaries yourself, there are many resources online that have already created them for you. In our context, we are interested in financial sentiment. [Here](https://sraf.nd.edu/textual-analysis/resources/#Master%20Dictionary) you can find the Loughran-McDonald financial sentiment dictionary. Using this resource, I created a formatted excel file that compiled this financial sentiment lexicon that I named the [Loughran-McDonald Master Dictionary.xlsx](https://github.com/pcann9/reddit-bitcoin-prediction/blob/master/resources/Loughran-McDonald%20Master%20Dictionary.xlsx) which you can download.

Using this resource, I created a csv called [additional_sentiment.csv](https://github.com/pcann9/reddit-bitcoin-prediction/blob/master/resources/additional_sentiment.csv) with a column for positive and negative words in the Loughran-McDonald dictionary that I edited using my discretion. To this csv, I also added domain specific sentiment words that represent the bitcoin community on reddit.

If you create your own additional sentiment dictionary, be weary of duplicate words in your dictionary. A useful tool to help with this is conditional formatting in excel which is why I recommend you first create/edit/modify it in excel and then export it to a format like CSV.

### 4. Combine Vader and Loughran-McDonald Lexicon
In this step, you are going to add [additional_sentiment.csv](https://github.com/pcann9/reddit-bitcoin-prediction/blob/master/resources/additional_sentiment.csv) to the [vader lexicon text file here](https://github.com/pcann9/reddit-bitcoin-prediction/blob/master/resources/vader_lexicon(real).txt). To do so:

```R
# import VADER lexicon txt file
library(readr)
vader <- read_delim("vader_lexicon.txt", "\t", escape_double = FALSE, col_names = FALSE, trim_ws = TRUE)

# format lexicon
vader$word <- vader$X1
vader$mean <- vader$X2
vader$sd <- vader$X3
vader$raw <- vader$X4
vader <- vader[ ,c("word", "mean", "sd", "raw")]

# import additional sentiment data
library(readr)
additional_sentiment <- read_csv("additional_sentiment.csv")
negative <- additional_sentiment$negative
positive <- additional_sentiment$positive

# remove NA from positive data (since positive is shorter than negative)
positive <- positive[!is.na(positive)]

# to lowercase
negative <- tolower(negative)
positive <- tolower(positive)


# words in common with VADER
neg_similar <- vader[vader$word %in% negative, ]
pos_similar <- vader[vader$word %in% positive, ]

# words not in VADER to add
neg_add <- negative[!negative %in% vader$word]
pos_add <- positive[!positive %in% vader$word]


# define weight of added negative/positive words (arbitarily chosen)
neg_weight <- -1
pos_weight <- 1

# create neg df
neg_df <- data.frame(matrix(NA, nrow = length(neg_add), ncol = 4))
colnames(neg_df) <- c("word", "mean", "sd", "raw")
neg_df$word <- neg_add
neg_df$mean <- rep(neg_weight, length(neg_df$mean))
neg_df$sd <- rep(0.5, length(neg_df$sd))
neg_df$raw <- rep("[-1, -1, -1, -1, -1, -1, -1, -1, -1, -1]", length(neg_df$raw))

# create pos df
pos_df <- data.frame(matrix(NA, nrow = length(pos_add), ncol = 4))
colnames(pos_df) <- c("word", "mean", "sd", "raw")
pos_df$word <- pos_add
pos_df$mean <- rep(pos_weight, length(pos_df$mean))
pos_df$sd <- rep(0.5, length(pos_df$sd))
pos_df$raw <- rep("[1, 1, 1, 1, 1, 1, 1, 1, 1, 1]", length(pos_df$raw))

# append pos/neg df with VADER
additional_sentiment <- rbind(neg_df, pos_df)
darth_vader <- rbind(vader, additional_sentiment)

# export data
write.table(darth_vader, file = "vader_lexicon.txt", row.names = FALSE, col.names = FALSE, quote = FALSE, sep = "\t", fileEncoding = "UTF-8")
```

The new VADER lexicon that I created is available [here](https://github.com/pcann9/reddit-bitcoin-prediction/blob/master/resources/vader_lexicon_fake.txt) for download.

### 5. Replace VADER Lexicon with our New Lexicon
Go to the same file location that we went to here `Python_File_Location>Lib>site-packages>vaderSentiment` and replace our newly created lexicon with the old one. They will have the same name so make sure not to get them mixed up. 

# *IMPORTANT:*
### In order for the exported data to work as VADER's lexicon, you need to open it in notepad. If you scroll all the way down to the last row in the csv, you will notice that there is an empty newline after the last row. This will cause VADER to crash and it will not be able to work. To make it work, delete this empty newline and then it will work. 

# -- End --
