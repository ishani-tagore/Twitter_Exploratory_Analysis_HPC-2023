## Analysing Twitter Data Related to the Education Domain #### 
In this and the remaining assignments for this class, I analyse the Twitter API dataset made available to me through a Data Science class at UChicago. My goal is to identify whether Twitter isa credible source of information, 
which reflects the emergence of important trends or topics in education. For Assignment 7 below, I have conducted preliminary EDA to 
identify relationships and patterns among variables, and understand what types of features drive popularity of a "Twitterer" or Topic in Education. 

In future assignments, we extend this analysis to conduct Topic Modeling on tweets and user descriptions (we have proxied relevant topics with hashtags for this assignment). 

# UPDATE! This respository documents Stage 1 of this analysis. 
Final results are <a href="https://docs.google.com/presentation/d/1ndK7mfd4J4IDGlGF304IG1O9RUZf2ADk/edit?usp=sharing&ouid=105860058275521700562&rtpof=true&sd=true">documented in these Google Slides</a>

 
# Discussion on Scalablity Implications: 
My approach has been to fail fast with code errors on small subsamples of the data, and then gradually scale up my code. 
These tweets are collected on the topics of education, but I have filtered for the hashtags related to primary, secondary or higher education: ['k12', 'highered', 'bookban', 'teachersunion', 'remoteteaching', 'remotelearning', 'schoolreopen']. 
We will analyse the full set of education-related tweets in the following assignments. 

When scaling up, I will: 
1. Work from parquet files: After conducting basic ETL and subsetting the data, I wrote the files to parquet in my cloud storage bucket, and proceeded to read from there in following sessions. For this graph-like Twitter data, Parquet offers the following benefits: 
> Columnar storage of Parquet files enables more efficient compression and better query performance as I include more Tweet records into the analysus
> Schema evolution: Parquet files have built-in support for schema evolution, allowing me to add, remove, or modify columns without rewriting the entire dataset. This flexibility is particularly useful for Twitter data which has a vast, rich and nested schema which must neccesarily be loaded piecemeal for fast analysis. 
> Predicate pushdown is supported. When executing queries, only relevant columns and rows are read from the storage, minimizing data movement and improving query performance.

2. Consider domain partitioning: 
For this assignment, I have restricted myself to education tweets and filtered on specific hashtags to reduce the size of the problem. Random sampling, which works with other big datasets, would not have worked here because of the interconnectedness of the data (tweets, retweets, follows, mentions, likes). 

Going forward, I will use MLIB's topic modeling to partition the data into Twitter account types (e.g. non profits, govts, influencers and businesses) and topic types (politics & poicy, edtech, other) in order to understand if and where credible information on emerging education topics exists on Twitter. 

Many problems of interest involve looking at the top decile of retweets or acitvity vs the rest - another potential partition and parallization option. Lastly, agressively grouping and aggregating the data before plotting has been a time-saver.

3. Wherever possible, I have refrained from moving between RDD, Pyspark and especially pandas.

4. Make strategic use of caching: Caching of objects to be immediately used in plotting has been a time-saver. Deliberate caching prior to Pyspark sql statement execution might reduce time as well .

### Time period & Schema of the sampled Twitter data
The time period: 5th April, 2022 - 25th February, 2023
root
 |-- date_created: timestamp (nullable = true) <br>
 |-- user_name: string (nullable = true) <br>
 |-- user_id: long (nullable = true) <br>
 |-- user_description: string (nullable = true) <br>
 |-- followers: long (nullable = true)  <br>
 |-- user_location: string (nullable = true) <br>
 |-- hashtag: string (nullable = true) <br>
 |-- tweet_id: long (nullable = true) <br>
 |-- tweet_text: string (nullable = true) <br>
 |-- retweeted_from: string (nullable = true) <br>
 |-- mentions_g: array (nullable = true) <br>
 |    |-- element: struct (containsNull = true) <br>
 |    |    |-- id: long (nullable = true) <br>
 |    |    |-- id_str: string (nullable = true) <br>
 |    |    |-- indices: array (nullable = true) <br>
 |    |    |    |-- element: long (containsNull = true) <br>
 |    |    |-- name: string (nullable = true) <br>
 |    |    |-- screen_name: string (nullable = true) <br>
 |-- retweeted_encoded: integer (nullable = true) <br>
 |-- tweet_count: long (nullable = true) <br>
 
### Original Content: Who are the top 10 most activer Twitterers in Education?
In our sample of education related tweets, the most active Twitterers posting <ins>original content</ins> include individual infuencers like Adam Sanford and Dr. Karen Connaghan who post about teaching techniques and edtech tools, posting from unverified accounts. Interestingly, the Clark College Job Board is the second most active producer of original content in our sample, indicating that some original content can be noice. 

 ![alt text](https://github.com/macs30113-s23/a7-ishani-tagore/blob/a71cd20e7b21e456742d6870713a90682c5ad9a9/Active_Twits_HPC2.png)
 
### Are most tweets original content or just copies of the original tweets / retweets?
Non-verified accounts both create ~20x the amount of original content as blue-verified accounts, and ~3x the amount of retweets. One would have expected news journalism accounts, non-profits and government accounts to create original content, posting news and generating press. This might still be the case, but from the chart below, it seems 'verified_status' is not a good identifier of accounts like these. I will try PySpark topic modeling on the user_descriptions to categorize org_type going forward.
![alt text](https://github.com/macs30113-s23/a7-ishani-tagore/blob/947813f5ca3c102b3731d66e4f83cd2c826ed5df/ORIGINALITY_HPC5.png)


 ### Who are the most followed Twitterers in Education?
While individual influencers are the most active tweeters, non-profits, edtech magazines and edtech-focused authors are the most followed. This also indicates that edtech might be the most followed topic. In the future, we will see if this trend holds up when segmenting the data by influencers, verified govt accounts, and other verified accounts (people and businesses).
 ![alt text](https://github.com/macs30113-s23/a7-ishani-tagore/blob/a71cd20e7b21e456742d6870713a90682c5ad9a9/Followed%20Twits_HPC1.png)
 
 ### How is the 'Number of followers' distrubuted across the users?
The data here is in line with the power law, where a majority of Twitter accounts have less than 12.5K followers, and a fraction of accounts are "influential" in attracting followers. This provides potential thresholds we could use to tag influential accounts. Excluding verified accounts, which I am assuming our specific sample of tweets has missed, non-verified accounts with more than 25K followers could be analysed as "influential". In further analyses, I would want to know: What has more reach? The many users with few followers (perhaps running niche audiences) or the right side of this graph - the few influential users, with millions of followers (but at least 12.5K followers)? 
 
 ![alt text](https://github.com/macs30113-s23/a7-ishani-tagore/blob/aafaa016d63b44f71f08cd1fac40f07138319340/Power_Law_Followes_HCP4.png)
 
 ### Who gets @mentioned the most?
The top one/third most mentioned accounts are also, by far, the most active Twitters. This group averages 100K tweets during this approximately one year time period, while the rest of the sample averages between 15K-20K annual tweets. Surprisingly, the bottom third of mentioned Twitters are slightly more active than the middle group. In the futrure, analysing the type of organizations or people would be insightful: Are news events, for example, characterized by mentions of political personas? Does mentioning a known influencer drive the reach of a tweet? 
being most frequently mentioned would provide insight into whether a tweet reflects a news event, a long-trending topic or 
  ![alt text](https://github.com/macs30113-s23/a7-ishani-tagore/blob/1847d724a4330cb36f72a5ff0107df483d085086/Mentions_Tweets_HCP3.png)
 
 # Future direction of analysis
 In the following assignments, we will use Graphframes, Pyspark MLIB's Topic Modeling and Jaccard Simmilarity to answer the questions: 
 
1. Is the content mostly unique? Or is it usually people copy-pasting the same text? Does the pattern of text similarity vary between official verified accounts and non-official accounts?  

2. Who is retweeting who? Who is the origin of the original content? We will use Pyspark Graphframes to find the nodes
(original tweeters) and edges (retweeters) for a politicized topic (Black History Courses & critical race theory) and an influencer-driven topic (edtech solutions), and a neutral education topic (pandemic-driven test score declines)

3. What topics are dominating the conversation among verified govt accounts? among other verified accounts? among influencers (non-verified accounts with more than 25K followers)? 

 
 
 
 
