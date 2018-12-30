# Cleaning Reddit Data in R

### 1. Import Data to R
After downloading the files from your bucket onto your computer, move them to your R working directory so we can then bring our data into R. In order to do so:  
```R
library(readr)
data <- read_csv("my_data_all_even.csv")
```

### 2. (Optional but Suggested) Subset Data to Work With 
The size of our data is large for R to process and it can be slow. In order to tweak, edit, and finalize our data cleaning process, we are going to work with a subset of our data to do such. In order to do so:
```R
sample.id <- seq(from = 5, to = length(data$author), by = 20)
data.sample <- data[sample.id, ]
```
Our `sample.id` is a numeric vector where it is a sequence of numbers from 5 to the number of rows of `data` where it counts by 20. So starting at 5, the next number would be 25 then 45, etc. until it reaches the number of rows in `data` which for me is roughly 1.7 million. Then, we define a new dataframe called `data.sample` that is defined as the rows of `data` that are in `sample.id`. So, this means that having the "by" argument in `seq` as 20 gives us 1/20th of our data in `data`.
- *This step is optional. For future code below, just replace `data` with `data.sample`*

### 3. Filter out Bots
 We want to filter out bots becuase we are only interested in reddit comments that are made by real people. In addition, bot posts are very redundant and can interfere with our analysis. [Here](https://www.reddit.com/r/autowikibot/wiki/redditbots) is a list of some bots on reddit. However, this list is dated and does not include many bots that are still found in our data. I manually went through the data and created [reddit_bot_names.csv](https://github.com/pcann9/Predict_Bitcoin_Using_Reddit_Sentiment/blob/master/resources/reddit_bot_names.csv) , a new appended list which includes many of these bots that were missed. To filter them out:
```R
# import csv of bot usernames
library(readr)
reddit_bot_names <- read_csv("reddit_bot_names.csv")

# redefine bot usernames as list
reddit_bot_names <- as.list(reddit_bot_names$bot_names)

# redefine "data" without bot usernames as author of posts
data <- data[!data$author %in% bot_list, ]
```
This code takes the character vector of bot usernames and converts it into the list `bot_list`. We use this list to subset `data` where we will not include observations where `data$author` matches any of the patterns in `bot_list`. This gives us a new dataframe without any bots. 

### 4. Remove NA rows
The data has some observations that return "NA". I think this is due to data that is returned from banned subreddits (such as r/incel). We don't want this data (especially comments from r/incel) as some of the steps in our analysis require there to be no "NA" data. To do so:
```R
#See if there are any NA
sapply(data, function(x) sum(is.na(x)))

# define rows with NA
na.rows <- apply(data, 1, function(x){any(is.na(x))})

# redefine df without NA rows
data <- data[!na.rows,]

# check to see if it worked
sapply(data, function(x) sum(is.na(x)))
```

### 4. Clean Text Data
The comment data contains things like urls, references to other subreddits, other users, and raw comment text that is used to format comments on the website. To get rid of all of this:
```R
# filter out "http|https|www" links
data$body <- gsub('(http|https|www)[^([:blank:]|\\"|<|\n\r)]+\\w', " ", data$body)

# url that starts with domain name "reddit.com" 
data$body <- gsub('[[:alnum:]_-]+(\\.com)[^([:blank:]|\\"|<|\n\r)]*', " ", data$body)

#filter our usernames in comments
data$body <- gsub('(/u/|u/)[[:alnum:]_-]+', " ", data$body)

#filter out subreddit in comments
data$body <- gsub("(/r/|r/)[[:alnum:]_-]+", " ", data$body)

# filter out odd character combos
data$body <- gsub("&gt;", " ", data$body) 

# filter out control characters
data$body <- gsub("[[:cntrl:]]", " ", data$body)

# filter out numbers
data$body <- gsub("[[:digit:]]", "", data$body)

# keep the symbols that you do want
data$body <- gsub("[^[:alnum:][:blank:]?.!,'_-]", "", data$body)
```
### 5. Language Filtering
Some of the reddit comments are in languages other than english. Because of this, we may want to filter them out as our sentiment classifier won't be able to work on them. There are a few different ways of doing this, the first of which you can do in R below. There are more methods available in the next section.

##### > method 1: filter via subreddit
First, you will need to create a csv of non-english subreddits to filter out. I have manually created one called [nonenglish_subreddits.csv](https://github.com/pcann9/Predict_Bitcoin_Using_Reddit_Sentiment/blob/master/resources/nonenglish_subreddits.csv) which you can use. How we filter out these subreddits is similar to how we filtered out the bots via their username above. To do so:
```R
# import packages
library(readr)

# our csv of non-english subreddits to filter out
nonenglish_subreddits <- read_csv("nonenglish_subreddits.csv")

# english data
english <- data[!data$subreddit %in% nonenglish_subreddits$nonenglish_subreddits, ]

# non-english data
non-english <- data[data$subreddit %in% nonenglish_subreddits$nonenglish_subreddits, ]
```
You should always check your non-english data to see if it is truly a good filter. In some instances, you may accidentally filter out real english data and it is up to you to determine if that tradeoff is worth it.

### 6. Export Data as CSV
After cleaning our data, we may want to export it for further analysis in other programs. To do so:
```R
# export data as csv
write.csv(x = data, file = "cean_data_even.csv", row.names = FALSE, fileEncoding = "UTF-8")
```
#### repeat this process with `my_data_all_odd.csv`
# -- End --
