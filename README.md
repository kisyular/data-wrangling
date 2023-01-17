# Introduction

Real-world data rarely comes clean. Using Python and its libraries, you will gather data from a variety of sources and in a variety of formats, assess its quality and tidiness, then clean it. This is called data wrangling. You will document your wrangling efforts in a Jupyter Notebook, plus showcase them through analyses and visualizations using Python (and its libraries) and/or SQL.

The dataset that you will be wrangling (and analyzing and visualizing) is the tweet archive of Twitter user [@dog_rates](https://twitter.com/dog_rates), also known as [WeRateDogs](https://en.wikipedia.org/wiki/WeRateDogs). WeRateDogs is a Twitter account that rates people's dogs with a humorous comment about the dog. These ratings almost always have a denominator of 10. The numerators, though? Almost always greater than 10. 11/10, 12/10, 13/10, etc. Why? Because ["they're good dogs Brent."](http://knowyourmeme.com/memes/theyre-good-dogs-brent) WeRateDogs has over 4 million followers and has received international media coverage.

WeRateDogs [downloaded their Twitter archive](https://support.twitter.com/articles/20170160) and sent it to Udacity via email exclusively for you to use in this project. This archive contains basic tweet data (tweet ID, timestamp, text, etc.) for all 5000+ of their tweets as they stood on August 1, 2017. More on this soon.

If you need to run this project in your machine, please install the following libraries:

```
    - pandas
    - NumPy
    - requests
    - tweepy
    - json
```

## Project Motivation

### Context

Your goal: wrangle WeRateDogs Twitter data to create interesting and trustworthy analyses and visualizations. The Twitter archive is great, but it only contains very basic tweet information. Additional gathering, then assessing and cleaning is required for "Wow!"-worthy analyses and visualizations.

### The Data

In this project, you will work on the following three datasets.

### Enhanced Twitter Archive

The WeRateDogs Twitter archive contains basic tweet data for all 5000+ of their tweets, but not everything. One column the archive does contain though: each tweet's text, which I used to extract rating, dog name, and dog "stage" (i.e. doggo, floofer, pupper, and puppo) to make this Twitter archive "enhanced." Of the 5000+ tweets, I have filtered for tweets with ratings only (there are 2356).

## Gathering Data

### Enhanced Twitter Archive

I have downloaded this file manually by clicking the following link: [twitter_archive_enhanced.csv](https://d17h27t6h515a5.cloudfront.net/topher/2017/August/59a4e958_twitter-archive-enhanced/twitter-archive-enhanced.csv). I then have imported this file into a pandas DataFrame at the very beginning of my wrangling efforts.

```python
import pandas as pd
import numpy as np
import requests
import os
twitter_archive = pd.read_csv('./twitter-archive-enhanced/twitter-archive-enhanced.csv')
```

### Image Predictions File

The tweet image predictions file was hosted on Udacity's servers and was downloaded programmatically using the requests library. This file was read into a pandas DataFrame and was assessed visually and programmatically for quality and tidiness issues.

```python
folder_name = 'image_predictions'
if not os.path.exists(folder_name):
    os.makedirs(folder_name)

image_predictions = requests.get('https://d17h27t6h515a5.cloudfront.net/topher/2017/August/599fd2ad_image-predictions/image-predictions.tsv')
with open(os.path.join(folder_name, image_predictions.url.split('/')[-1]), mode='wb') as file:
    file.write(image_predictions.content)

image_predictions = pd.read_csv('image_predictions/image-predictions.tsv', sep='\t')
```

### Additional Data via the Twitter API

I have queried the Twitter API for each tweet's JSON data using Python's Tweepy library and stored each tweet's entire set of JSON data in a file called `tweet_json.txt` file. Each tweet's JSON data is written to its own line. Then I have read this .txt file line by line into a pandas DataFrame with tweet ID, retweet count, and favorite count.

> **TIP**: It is important to note that the following code was provided by Udacity and was used to query the Twitter API for each tweet's JSON data using Python's Tweepy library and to store each tweet's entire set of JSON data in a file called tweet_json.txt file. Each tweet's JSON data was written to its own line. Then this tweet JSON was read line by line into a pandas DataFrame with tweet ID, retweet count, and favorite count.

```python
import tweepy
import json
import time

# Twitter API Keys from keys.txt
with open('keys.txt', 'r') as file:
    consumer_key = file.readline().strip()
    consumer_secret = file.readline().strip()
    access_token = file.readline().strip()
    access_secret = file.readline().strip()
# Query Twitter API for each tweet in the Twitter archive and save JSON in a text file
# These are hidden to comply with Twitter's API terms and conditions

# Store each tweet's returned JSON as a new line in a .txt file
# Query the Twitter API for each tweet's JSON data using Python's Tweepy library and store each tweet's entire set of JSON data in a file called tweet_json.txt file.
auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
auth.set_access_token(access_token, access_secret)

api = tweepy.API(auth, wait_on_rate_limit=True)

# Query Twitter's API for JSON data for each tweet ID in the Twitter archive
count = 0
fails_dict = {}
start = time.time()
# Save each tweet's returned JSON as a new line in a .txt file
with open('tweet_json.txt', 'w') as outfile:
    # This loop will likely take 20-30 minutes to run because of Twitter's rate limit
    for tweet_id in twitter_archive['tweet_id']:
        count += 1
        print(str(count) + ": " + str(tweet_id))
        try:
            tweet = api.get_status(tweet_id, tweet_mode='extended')
            print("Success")
            json.dump(tweet._json, outfile)
            outfile.write('\n')
        except tweepy.errors.TweepyException as e:
            print("Fail")
            fails_dict[tweet_id] = e
            pass
end = time.time()
print(end - start)
print(fails_dict)
```

## Assessing Data

The data was assessed visually and programmatically for quality and tidiness issues. The quality issues were defined as issues with content. This included missing data, inaccurate data, and duplicate data. The tidiness issues were defined as issues with structure. This included issues with the structure of the data that made it difficult to analyze and visualize. **It is imperative to note that not all data issues tackled in the `wrangle_act.ipynb` are mentioned here in this report.** The data was assessed as follows:

### 1. The WeRateDogs Twitter archive

The WeRateDogs Twitter archive was assessed visually and programmatically for quality and tidiness issues. The following issues were identified:

#### Quality

-   **Missing data** in the following columns: `in_reply_to_status_id`, `in_reply_to_user_id`, `retweeted_status_id`, `retweeted_status_user_id`, `retweeted_status_timestamp`.
-   **Incorrect data or invalid data** The rating denominators are not all 10.
-   **Incorrect data types** (object instead of datetime) in the following columns: timestamp
-   **incorrect data values** The dog names are not all correct, they contain "a", "an", "the", "None", etc.

#### Tidiness

-   **Data spread accross columns** The dog stages are in four different columns instead of one column, `doggo`, `floofer`, `pupper`, and `puppo`

### 2. The tweet image predictions

The tweet image predictions file was assessed visually and programmatically for quality and tidiness issues. The following issues were identified:

#### Quality

-   **Data inconsistency** the `p1`, `p2`, and `p3` columns are not all lowercase.
-   **Data inconsistency** the `p1`, `p2`, and `p3` columns contain non-dog names, such as "web site", "bookcase", "desktop computer", etc.

#### Tidiness

-   The columns `p1`, `p1_conf`, `p1_dog`, `p2`, `p2_conf`, `p2_dog`, `p3`, `p3_conf`, and `p3_dog` in the `image_predictions` table are not descriptive.

**Visual Assessment** provided me with opportunities to identify issues with the data. I was able to notice that in `twitter_archive` data, dog stages were spread across four columns instead of one column. I was also able to notice that in `image_predictions` data, the columns `p1`, `p1_conf`, `p1_dog`, `p2`, `p2_conf`, `p2_dog`, `p3`, `p3_conf`, and `p3_dog` were not descriptive.

**Programmatic Assessment** provided me with better opportunities to identify more issues with the three datasets. I was able to notice that in `twitter_archive` data, the dog names were not all correct, they contained "a", "an", "the", "None", etc. I was also able to notice that in `image_predictions` data, the `p1`, `p2`, and `p3` columns contained non-dog names, such as "web site", "bookcase", "desktop computer", etc. These issues were not visible in the visual assessment. Moreover, to tackle these issues, I divided the data into two categories: quality and tidiness.

## Cleaning Data

In each dataset, the issues were defined, coded, and tested. This process made it easier to understand how data wrangling is done. In each of the dataset and the issue I resolved, I have included a sample of the code used to resolve the issue.

The first step in cleaning the data was to make copies of the three datasets. This was done using the `copy()` method. The following code was used to make copies of the three datasets:

```python
twitter_archive_clean = twitter_archive.copy()
image_predictions_clean = image_predictions.copy()
tweet_json_clean = tweet_json.copy()
```

### 1. The WeRateDogs Twitter archive

I changed the data type of the `timestamp` column from object to datetime. I used the `to_datetime()` method to change the data type of the `timestamp` column from object to datetime.

```python
twitter_archive_clean['timestamp'] = pd.to_datetime(twitter_archive_clean['timestamp'])
```

I also dropped the columns `in_reply_to_user_id`, `retweeted_status_user_id`, and `retweeted_status_timestamp` because they contained missing data. I used the `drop()` method to drop the columns

```python
twitter_archive_clean = twitter_archive_clean.drop(['in_reply_to_user_id', 'retweeted_status_user_id', 'retweeted_status_timestamp'], axis=1)
```

As mentioned earlier, some rating denominators were not 10. I used the `loc()` method to select the rows where the rating denominators were not 10. I then filled the rating denominators with 10.

```python
twitter_archive_clean.loc[twitter_archive_clean['rating_denominator'] != 10, 'rating_denominator'] = 10
```

Moreover, I noticed that the dog names were not all correct, they contained "a", "an", "the", "very", "just", "quite", "one", "getting" "actually", "mad", etc. On close inspection, I noticed most of the dog names were lowercase. I used the `loc()` method to select the rows where the dog names were lowercase. I then using regular expressions, I replaced the lowercase dog names with names that were extracted from the text column.

```python
twitter_archive_clean.loc[twitter_archive_clean['name'].str.islower(), 'name'] = twitter_archive_clean['text'].str.extract('(?:This is|Meet|Say hello to|named|name is|Here we have) ([A-Z][a-z]+)', expand=True)
```

There were more issues in the `twitter_archive` dataset. However, not all issues which were identified in the assessment and resolved in the cleaning process in the `wrangle_act.ipynb` are mentioned here in this report.

### 2. The tweet image predictions

I used the `loc()` method to select the rows where the `p1`, `p2`, and `p3` columns were not all lowercase. I then used the `str.lower()` method to convert the `p1`, `p2`, and `p3` columns to lowercase.

```python
image_predictions_clean['p1'] = image_predictions_clean['p1'].str.lower()
image_predictions_clean['p2'] = image_predictions_clean['p2'].str.lower()
image_predictions_clean['p3'] = image_predictions_clean['p3'].str.lower()
```

Using the `p1_dog`, `p2_dog`, and `p3_dog` columns, I selected the rows where the dog breed predictions were either true. All other rows were dropped.

```python
image_predictions_clean = image_predictions_clean.query('p1_dog == True or p2_dog == True or p3_dog == True')
```

### 3. The tweet JSON data

In the generated `tweet_json` dataset, I selected the columns `id`, `retweet_count`, and `favorite_count`. Other columns were not needed or redundant, hence they were dropped.

```python
tweet_json_clean = tweet_json_clean[['id', 'retweet_count', 'favorite_count']]
```

> **TIP**: It is important to note that not all issues which were identified in the assessment and resolved in the cleaning process in the `wrangle_act.ipynb` are mentioned here in this report. Moreover, this report only contains the `code` part of cleaning process. The `define` and `test` parts of the cleaning process are not included in this report.

## Storing

After intensive data wrangling, I renamed the following columns in the `image_prediction` `p1`, `p1_conf`, `p1_dog`, `p2`, `p2_conf`, `p2_dog`, `p3`, `p3_conf`, and `p3_dog` into `prediction_1`, `prediction_1_confidence`, `prediction_1_is_dog`, `prediction_2`, `prediction_2_confidence`, `prediction_2_is_dog`, `prediction_3`, `prediction_3_confidence`, and `prediction_3_is_dog`. I used the `rename()` method to rename the columns.

```python
image_predictions_clean = image_predictions_clean.rename(columns={'p1': 'prediction_1', 'p1_conf': 'prediction_1_confidence', 'p1_dog': 'prediction_1_is_dog', 'p2': 'prediction_2', 'p2_conf': 'prediction_2_confidence', 'p2_dog': 'prediction_2_is_dog', 'p3': 'prediction_3', 'p3_conf': 'prediction_3_confidence', 'p3_dog': 'prediction_3_is_dog'})
```

Moreover, I noticed all three datasets can be merged into one dataset. I used the `merge()` method to merge the three datasets into one dataset. I then stored the merged dataset in a CSV file named `twitter_archive_master.csv`.

```python
twitter_archive_master = pd.merge(twitter_archive_clean, image_predictions_clean, on='tweet_id', how='inner')
twitter_archive_master = pd.merge(twitter_archive_master, tweet_json_clean, left_on='tweet_id', right_on='id', how='inner')

# Store the master dataset in a CSV file
twitter_archive_master.to_csv('twitter_archive_master.csv', index=False)
```

## Analyzing and Visualizing

After storing the master dataset in a CSV file, I loaded the CSV file into a dataframe. I used the `read_csv()` method to load the CSV file into a dataframe.

```python
twitter_archive_master = pd.read_csv('twitter_archive_master.csv')
```

For my analysis, I was curious to know the following questions:

1. What is the most common dog stage?
2. What is the most common dog breed?
3. Is there a relationship between the retweet count and the favorite count?
4. Average retweet count and favorite count over time
5. How does the most common dog breed look like?

## What is Next?

I encourage the reader to download or clone this repository and explore the data.
