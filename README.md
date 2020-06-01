### Capstone-Project-Internet-Forum-Mental-Health-Risk-Prediction
Is it possible to identify internet forum users who might require support for mental health related issues based on their wider interests? 

In this project I used Reddit post data from all users in 2018 to predict whether a user is likely to post to mental health advice forums (forums are called 'subreddits') based on user activity/posting in other forums.

### Overall Summary

This was my capstone project undertaken for completion of the General Assembly Data Science Immersive Programme.

My goals for the project were to understand and predict mental health risk in users of the internet forum website Reddit, initially I wanted to estimate the impact of news topics and sentiment on mental health risk, before pivoting to look at user interests and the relationship with mental health risk. 
 * Mental heatlh risk in this context is whether a user posts to the various mental health related subreddits
 * Users of these subreddits are typically looking to vent or to ask for advice from fellow users

Initially I wanted to look a potential time series analysis of the overall number of posts to mental health related subreddits based on sentiment & clustering of daily news stories from the major Reddit news forum. Unfortunately data was restricted to news headlines only and after some initial EDA it became apparent that more information per article would be required to perform adequate NLP and create a worthwhile analysis.

I then decided to pivot into looking into user behaviour in terms of posting to other, unrelated, forums and whether this can be used to predict mental health risk. I created a boolean target variable of whether a user had posted to a mental health related subreddit over the time period as my estimator for mental health risk in a specific individual.

### Problem Statement & Project Goals 
**Background**
* The forum website reddit hosts a wide variety of different themed forums (subreddits), including a large number of mental health dedicated subreddits
* The goal of these subreddits is to provide users support and information relating to various topics relevant to mental health
* Understanding the link between mental health dedicated subreddits and seemingly unrelated subreddits could help websites like Reddit identify users who may be suffering in terms of mental health and who could benefit from promotion of advice forums and other resources which offer support

**Hypotheses & Goals**
* This projects seeks to identify internet forum users who may be at risk of mental health disorders based upon their use of other internet forum types
* The underlying hypothesis states that there is a link between mental health and wider interests

**Problem Statement**
* Identification or diagnoses of at risk patients from a wider population is a key goal for many machine learning applications
* However, for the majority of disorders only a small minority of the sample will be positive for the disorder we wish to identify
* Additionally a significant proportion of a sample may be unidentified as at risk
* Applying standard ML techniques is likely to lead to models to ignoring the minority class, resulting in low to no identification of these individuals
* A key component for this project is how to extract the most relevant information from a severely imbalanced sample

### Data Collection & Cleaning 
 * File: MKII_Data_Extraction_Cleaning
 
**Data Collection**
* Data was pulled from the pushshift.io dataset stored on BigQuery via SQL
* The data covered 2018 and included post author and the subreddit posted to, as well as the count of total posts per subreddit per author 
* In cases where the author or subreddit were deleted these were excluded from the SQL query
  * Due to dataset size the pull was split into three sections and then combined via joining
  * Post-joining any null values were removed
  * Author names were converted to numbers to both save memory and avoid delving into specific author behaviour on reddit
* Before any further cleaning steps were taken there were 12.1M unique users, posting to 1.4M unique subreddits with a total of 125M posts in 2018:
![Unique features precleaning](data:image/png;base64,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HACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAIKEUAW57b9uzhoeHB90UAAAmhBQBHhFnRsSMoaGhQTcFAIAJIUWAAwCAkQhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEgoRYDb3tv2rOHh4UE3BQCACSFFgEfEmRExY2hoaNBNAQBgQkgR4AAAYCQCHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhFIEuO29bc8aHh4edFMAAJgQUgR4RJwZETOGhoYG3RQAACaEFAEOAABGIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASWnnQDRikqZ8+a9BNQB/NPWrPQTcBAJYb9sABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAICECHACAhAhwAAASIsABAEiIAAcAIKHl/lWqtjeQ9Mu2SatJ2lTSehExf3m3BwCAjJZ7gEfE7ZK2bT22/WlJOxPeAAD0rqchdNsb2v6m7YttP2Q7bE/tUnYj2z+zPWz7ftun2t54lOrfJ+n74286AACTV6/HwDeXdICkBZIu7FbI9qqSzpO0laSDJL1H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)
