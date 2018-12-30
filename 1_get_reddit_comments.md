# Reddit Comments

In order to get reddit comments off of Google BigQuery, you first need to sign in to the Google Cloud Platform which you can do at **[this](https://console.cloud.google.com)** link. You might need to have a google gmail account (which you should anyways) but I'm not sure if you can sign in with a non-gmail account. A point to note is that Google Cloud Platform can be confusing to use and hard to understand which is why I am noting every step to take in detail. I am also using the new Google Cloud Platform UI instead of the old one which can also be confusing. 

### 1. Make a project
A popup should be on the dashboard that enables you to make a project. Click create and I will call it `my-project`. 


### 2. Go to BigQuery
There is a scroll menu on the left-most side of the screen. This shows all the different tools that are included in the Google Cloud Platform. You will find BigQuery on the scrolldown menu on the left which is under a header that is called Big Data. Click BigQuery and continue. 


### 3. Create Dataset
At the top right hand corner, there is a button that says "Go to Classic UI". Click it and it will take you to the BigQuery classic UI. The reason why you do this is because 1)its easier to see all the available datasets 2)our reddit comments dataset isn't in "public datasets". In the classic UI, your project name, which in this case is `my-project` will be on the left. there is a dropdown arrow next to it that you use to create a new dataset which I will name `my_dataset`.


### 4. Running Queries to get Data
Under "fh-bigquery" you will find a dataset called "reddit_comments" which is the dataset that we will be using. You will enter the following script:
```SQL
#standardSQL  
SELECT author, subreddit, created_utc, score, controversiality, body  
FROM `fh-bigquery.reddit_comments.2009`  
WHERE REGEXP_CONTAINS(body, r'(?i)\b bitcoin\b')
```
*You should only do one/a few queries at a time. If you do too many, your query will fail as it will take up too much memory*

This code sets standard SQL as our scripting language. It SELECTs the variables that we would like to collect FROM the dataset that we specified of observations that contain " bitcoin" in the variable "body". In otherwords, this specific code gets us all comments from 2009 that contain the word " bitcoin". Note that bitcoin has a whitespace before it which is because we don't want comments with "bitcoin" as a part of a hyperlink. 

After you have run the query, click "Save as Table" and I named the data from the query above `my_data_2009`. Do this for every time period after 2009 as that was the year that bitcoin was created. For me, the last time period on BigQery is 8/2018


### 5. Before You can Export Data
You have to sign up for a free trial of Google Cloud Platform before you can export your data. If your dataset is small enough, you can just download it as a csv or JSON. That is not the case for us. **You will have to give valid credit card information** but they won't charge you without your consent and will give you $300 credit which lasts for one year. They don't secretly charge you after the trial runs out so its totally worth it. For this entire project, it costed me **$0.67 of the $300 credit** so just do it. 


### 6. Create a Bucket
After starting your free trial, go to the main Google Cloud Platform, go to the scrollable menu on the left, find the header "STORAGE" and click "Storage" click "CREATE BUCKET". Since buckets have to have globally unique names (I am jealous of the person who has "my_bucket"), I will name my bucket `myself_bucket`. In this bucket, create a folder which I will call `myself-bucket-folder`. This is the are where your data will be exported to from BigQuery.


### 7. Prepare Data for Export
After all of your queries are done loading, you are going to combine all of your data into one main dataset. Iterate the following code over all of your data do do such. I will name it `my_data_all`
```SQL
#standardSQL
SELECT * FROM `my-project.my_dataset.my_data_2009`
UNION ALL
SELECT * FROM `my-project.my_dataset.my_data_2010`
UNION ALL
SELECT * FROM `my-project.my_dataset.my_data_2011`
ORDER BY created_utc ASC
```
`my_data_all` is too big to be exported directly to `myself_bucket`. Our `my_data_all` exceeds the 2gb limit of exports to buckets. In order to get around that, we can split `my_data_all` in half. Since created_utc is unix time, there is a roughly equal likelihood that the timestamp is even or odd, this means we can parse our data in half using a modulo operation. Any even number modulo 2 is going to be 0 because even numbers are divisible by 2. Odd numbers are the exact opposite. Knowing this, we can create a dataset of even and odd timestamps roughly parsing it in half. Hypothetically, if our dataset was even bigger, we could use regular expressions to pattern match the last number of unix time which is 0-9 which would give us the ability to parse our data into roughly 10 equal sizes.  

This code will create the even half which I will call `my_data_all_even`
```SQL
#standardSQL
SELECT *
FROM `my-project.my_data_set.my_data_all`
WHERE ( created_utc % 2 ) = 0
```
This code will create the odd half which I will call `my_data_all_odd`
```SQL
#standardSQL
SELECT *
FROM `my-project.my_data_set.my_data_all`
WHERE NOT ( created_utc % 2 ) = 0
```

### 8. Exporting Data
After your data is small enough to be exported, click on `my_data_all_even` and then click CSV (or JSON if you want) and then the option "export". This will ask you for the "Google Cloud Storage URI". This is the filepath to your bucket>bucket folder>dataset name. Since "gs://" is already entered, I will enter `myself-bucket/myself-bucket-folder/my_data_all_even`. I will do this again for the odd dataset which will be `myself-bucket/myself-bucket-folder/my_data_all_odd`. After that, I will have all my data in the `myself-bucket-folder`. In order to download the data onto you computer, you just go to the `myself-bucket-folder`, click both, and they will start downloading in CSV format onto your computer to then be analyzed, cleaned, etc. in R.

### 9. Clean out Bucket
Storing data in your bucket is the part of this project that will cost money/credit. After downloading your data onto your computer, I would advise you to delete the data in the bucket. It is very easy to forget about and this will use up all of your credits if it is stored for a long period of time. **So delete the data after downloading it!**

# -- End --
