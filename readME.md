# **Modeling CryptoPunks**

## Table of Contents

1. [Executive Summary](#1.-Executive-Summary)
2. [Data Collection](#2.-Data-Collection)
3. [Data Cleaning & Pre-Processing](#3.-Data-Cleaning-&-Pre-Processing)
4. [Modeling](#4.-Modeling)
5. [Evaluation & Analysis](#5.-Evaluation-&-Analysis)
6. [Conclusion & Recommendations](#6.-Conclusion-&-Recommendations)
7. [Next Steps & Future work](#7.-Next-Steps-&-Future-work)
8. [Citations](#8.-Citations)
9. [Acknowledgements](#9.-Acknowledgements)



## 1. Executive Summary

With the advent of blockchain technology, maintaining a ledger of transactions and events and verifying ownership of assets has never been more efficient, transparent, and secure. These conditions are prime breeding grounds for innovative financial products and services to flourish, especially as traditional financial markets quiver over the prime brokerage risk of T+2 settlement latencies given the recent Robinhood debacle. Blockchain technology allows for basically instant settlement latencies and poses the potential for completely revamping our entire traditional financial market. 

This technology is also extremely suitable for art markets. Along side the rapidly growing decentralized financial space, protocols and communities of artists and collectors are exploding because blockchain technologoy fixes a number of isses that traditional art markets have endured for ages. Art is easily stolen and counterfeited. Art sales are extremely illiquid and often require being in a physical location. And shipping costs and brokerage fees are large. Ethereum solves these issues by 1. issuing non-fungible tokens that represent digital ownership of an asset, 2. facilitating auctions across the entire Web3 internet, 3. reducing the entry to barrier by completely eliminating the need to be present, and 4. reducing fees.

The very first art project on Ethereum is called CryptoPunks which was created by Larva Labs - a project spearheaded by John Watkinson and Matt Hall. A CryptoPunk is a non-fungible token that represents ownership of an image with a generated character. There are 10,000 total punks and each is uniquely different. This means no two punks are identical. There are five types of punks, 87 different attributes, and each punks can have a range of zero to seven attributes. The combination of each of these characteristics contribute to a punks rarity. 

Since inception, CrypoPunk sales have reached about $93M in sales, and recently mainstream serial investors such as Gary Vaynerchuk and Mark Cuban have brought even more interest into these markets. 

For this project I want to model CryptoPunks and their sale prices.


## 2. Data Collection

The data regarding each CryptoPunk and its attributes was gathered from the community via the Larva Labs Discord channel. The data regarding each punk's sales history was gathered from the Larva Labs website. ([*source*](https://www.larvalabs.com/cryptopunks)). Below are data dictionaries for each data source. 

| Feature              | Type   | Description                         |
|----------------------|--------|-------------------------------------|
| Punk ID              | int64  | ID of punk (0-9999)                 |  
| Type                 | object | Male, Female, Zombie, Ape, or Alien |      
| Skin                 | object | Light, Mid, Dark                    |
| Attribute            | object | unique attribute                    |

| Feature              | Type   | Description                                                            |
|----------------------|--------|------------------------------------------------------------------------|
| Punk ID              | int64  | ID of punk (0-9999)                                                    |  
| Transaction Type     | object | Sold, Offer, Bid, Bid Withdraw, Offer Withdrawn, Transfer, or Claimed  |      
| Amount               | float  | Light, Mid, Dark                                                       |
| Date                 | object | unique attribute                                                       |



## 3. Data Cleaning & Pre-Processing

In addition to the typical data cleaning process, I use OneHotEncoder to encode all of my categorical variables. Given the encoded columns, I calculated a scarcity score, which is effectively the rareness of a punk and its set of attributes. 

Upon further exploration of the data, I learned that the data is incredibly balanced. After having a total of 70,000 data points from my initial dataset, the final dataframe for modeling had 8,000 observations, only 3,808 unique IDs, and female punks made up 70% of sales. 

The transaction data I gathered is also fragmented. Transaction types did not have an order ID associated with it, so engineering a matching engine will require either more data or further data engineering. In the meantime, I decided to focus solely on sold transactions and imputed a number of potential target columns based on the sale price, floor price, and/or 30 day moving average price. I was able to match these transactions by date and ID. Unfortunately, my model is unable to capture how the order flow of each auction affects the sale price of each punk.


## 4. Modeling

Due to the challenge of modeling imbalanced data, I implemented a number of different machine learning techniques and attempted to infer commonalities among coefficients from the best performers. Here are a list of each of the techniques I used along with brief descriptions.

Logistic Regression: Two separate binary classifications that models sales data for each punk and its attribute to predict if a punk will trade above or at/below floor price and if a punk will trade above or below the 30 day moving average of all punks. 

Random Forest Classification: Two separate binary classifications that modeled sales data for each punk and its attributes to predict if a punk will trade above or at/below floor price and if a punk will trade above or below the 30 day moving average of all punks. This model's cross validation score after grid searching hyperparameters performed the best. 

Random Forest Regression: A regression that models sales data for each punk and its attributes to predict how far a punk will trade from the floor price. 

Linear Regression: A regression that models sales data for each punk and its attrributes to predict how far a punk will trade from the floor price. 

XG Boost Classifier: Two separate binary classifications that modeled sales data for each punk and its attributes to predict if a punk will trade above or at/below floor price and if a punk will trade above or below the 30 day moving average of all punks. This model performed ok, but not that much better than our baseline case. 

Bagging Classifier: Two separate binary classifications that modeled sales data for each punk and its attributes to predict if a punk will trade above or at/below floor price and if a punk will trade above or below the 30 day moving average of all punks.

## 6. Evaluation & Analysis

<img src="./images/featureimportances.png" width="550" height="250">

Above are the results of my Random Forest Model where I modeled attributes to predict whether or not a punk traded above or at/below the market's floor price. The chart includes the model's most important features, its cross validation score, and its sensitivity score. 

The accuracy score is only 4% better than the baseline model's normalized prediction count. However, this model suffered the least from the bias-variance trade off. Further, while we cannot necessarily rely on the accuracy of the model, we can infer some information from its feature importances and with our judgement determine whether or not there might be some useful information. 

In addition, the confluence among these feature importances and the coefficients from the binary logistic regression and the feature importances of the XG Boosting model might give us more reason relevance to useful features. 

<img src="./images/logreg_coefficients.png" width="550" height="250">

<img src="./images/xgb_feature_importances.png" width="550" height="250">

Based on the charts, we can reasonably assert that Females will trade a premium prices, which makes sense given females are less frequent than males, if we assume males should theoretically set the floor price. Interpreting the coefficient and feature importances of the remaining values is less reliable because the top features for each model are also tend to be the most traded attributes. This suggests the model is biased to assign too much predictive power to features that are more frequent, which is a problem given the imbalanced data set. The results become reflexive because the trend in price is up. The more an attribute is traded, the more likely it will trade a premium. 

Having said that, one might conclude that because the model is not making any sense of the price of a punk and its features, then there aren't necessarily any rules to justify the premium of any punk and therefore the market will need to correct itself.

All in all, even after attempting to mitigate the imbalanced data (via bagging, feature selection, KMeans clustering, and DBSCAN clustering), we cannot draw any predictive insight from the data. However, we can observe the results and potentially use them as expectations for our own perspective of how valuable each of these punks are. 


## 7. Conclusion 

In conclusion, it is hard to draw meaningful decisions from the model. The data is too imbalanced to rely on. However, we can draw observations from what the models did produce. Here are a few examples that I view as meaningful takeaways: 

- Female punks are more likely to trade at a premium 
- Generally, the market will correct itself 
- Punks with two to three attributes are more likely to trade at a premium. 
- Cigarettes, albinos, and moles are more likely to trade at a premium. 

In other words, if I see a Female punk with a cigarette and a mole offered at floor, I should buy it. 

## 8. Next Steps & Future work

A large part of the challenge I had was not being able to match the order flow of the transactions. To fix this, I want to pull the same data from etherscan, which has much more comprehensive data, including the time of day of each transaction. 

These markets are still very nascent. More data will also really benefit any future endeavors. 

I also think it is prudent to run a regression on specific attributes that have been traded frequently, so that we can at least draw inference from what information is frequent enough. 

## 9. Acknowledgements

The inspiration of this project was driven from my deep curiosity and dedication for digital assets. While I was not able to build a model that yielded valuable results, working through the trials and tribulations of munging through data and trying to build a useful model was always the goal. 

General Assembly has been a great experience. While I completely underestimated the amount of time and energy this immersive required, I am leaving the course as a much more useful person with a much sharper skill set, which were my intentions for taking the course from the start.

I am extremely grateful for all of my instructors, Charlie Rice, John Hazard, Hov Gasparian, and all of the transient instructors and guest speakers along the way. I look forward to bringing these newfound skills into the future of blockchain technology. 