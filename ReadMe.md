This work is based on Spark. Please find source code and full results [here](https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/7965635341886967/262132380329106/5164190521743747/latest.html)

**1. Motivation:**

Recommendation systems are widely used today by big companies such as Amazon, Youtube, and Netflix for their Internet products. A good recommendation system significantly increases their business values. In this work, ALS is used to build a recommendation system based on data from [grouplens](https://grouplens.org/datasets/movielens/latest/) (Small: 100,000 ratings and 3,600 tag applications applied to 9,000 movies by 600 users. Last updated 9/2018.) 

**Data characteristics: **

A major data characteristic of training a recommendation system is high sparsity. Just consider how many purchased items you have ever rated. One thing to point out is that most products only have a small number of reviews, and a small number of products are heavily rated. Also, most ratings are written by the same small amount of users.

Another important data feature is that the number of movies is around 10 times larger than that of the users.

![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/user_dis.png)
![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/movie_dis.png)

**Why ALS?: **

Alternating Least Square (ALS) Matrix Factorization is an intermediary along the pathway of developing recommendation systems and is widely used by many companies. The key characteristic of this approach is a combination of good scalability and predictive accuracy. A detailed description can be found on [Standford CME323](http://stanford.edu/~rezab/classes/cme323/S15/notes/lec14.pdf) and [NETFLIX-recommender systems](https://datajobs.com/data-science-repo/Recommender-Systems-[Netflix].pdf)

**2. Implementation:**

**Step.1 Data Exploration and Observations**

The source data is relatively clean. Three data tables are complete with the only table (links.csv) having null values. This table is also not related to the major work here currently. We will focus on the ratings.csv table since it contains the most important information for data training, and then the movies.csv as it provides more perspectives on the movie's content for users.

As expected, extreme cases for both users and movies are observed as mentioned above. The 'long-tail' characteristic is clearly present as the spasticity of the “user vs. item” matrix is as high as 0.983. 

**Step.2 Training and model selection**

The parameters that need tuning are rank (how many latent features for each user and movie), regParam (regulation parameter), and maxIter (max iteration). Grid search and cross-validation are used. Final parameters are determined and then used for training the whole set of data.

**Step.3 Analysis**

To compare the actual data and the predicted data, the first step is to round the predicted score to 0.5. (Follow with table) Then as all ratings are plotted against the userId and movie Id, I found the graph too messy. (Follow with graph) Here, we are looking at the mean square error for 4 special cases (users rated most and least movies, and movies with most and least users) for a better understanding of the result. 

![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/rounded_ratings.png)

| MSE | User  | Movie |
| -- | -- | -- |
| Most | 0.578 | 0.571 |
| Least  | 0.748 | 0.021 |

In terms of users, the prediction for who are more active users is better than that for who are less active, which is intuitively right. However, in terms of movies, it is surprising to find out that the prediction for movies’s ratings with the most number of ratings is far worse than that for movies with only one rating. This is also not due to the effect of averaging, which is  shown by the figures below. 

![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/most_by_movies.png)
![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/least_by_movies.png)

Though counterintuitive, this is easy to understand. Let's consider what the matrix factorization actually does. It is trying to guess the 'most likely' k-dimension features for each movie (m) and user (n) based on an n by m matrix with labeled ratings. 

![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/Matrix-Factorization.png)

First, let's consider a special case where we have one 4-star rating from one user for one movie. There are infinite combinations of results that can ensure 0 error, even when k=1. 

Then for a 2x2 table:

| rating (1-5) | movie 1 | movie 2 |
| -- | -- | -- |
| user 1 | 3 | 4 |
| user 2 | 1 | 5 |

If we set k=1, normally we can have solutions without error. That means we certainly know the taste of each user and the feature of each movie. If we lose one rating, to guess the rating by using matrix factorization for this missing value is very risky or actually invalid, because again there are infinite combinations of results. Increasing k would also lead to similar conclusion.

Based on the analysis above, we can quickly imagine a 1xn or an nx1 table will display the same result. So ideally, we need a reasonable 'activity' of movies and users which will ensure a higher correlation among users and movies. 

Further, it means that the categories of hidden k 'features' of one movie is not necessarily totally equal to those of other movies. It means we can not think of each row of the movie matrix as a certain feature, but hopefully, as the data and the correlation get large enough, the effect of those concerns would lessen.

As already reflected by the observations above, when averaged by the number of ratings, the absolute error is lower. At the same time, the error converges as the number of ratings gets to around 25. We can see that although the less-rated movies may be fitted better, it's more unstable. Therefore, ideally, I would recommend 50 to be the minimal number of ratings for movie to get fair results. 

![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/error_by_movie.png)
![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/error_by_movie_1.png)

**Step.4 Applications**

We can provide recommendations for users based on this model and find similar users or movies according to the cosine correlation of their latent features.

Beyond that, we should also pay special to attention to the users who are not represented well by the model (Also true for movies). 

**3. Conclusion**

Using ALS-based Matrix Factorization, I built a movie recommendation systems. The data displayed high sparsity and unbalanced ratio between the number users and movies. The latter could be a major reason for the unusual results above. 

Considering the application, I would recommend setting 50 as the threshold for the number of ratings for an optimal prediction for movies. It infers that we should actively encourage people to rate those movies which do not have this many reviews so far. Also, for those less-represented users and movies by the model, companies may want to take action to acquire more data or at least build a separate model for them.
