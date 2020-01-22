This work is based on Spark. Please find source code and full results [here](https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/7965635341886967/262132380329106/5164190521743747/latest.html)

# Background

Recommendation systems are widely used today by big companies such as Amazon, Youtube, and Netflix for their Internet products. A good recommendation system significantly increases the business values of those products by targeting correct customers. The goal of this work is to build a movie recommendation system by Alternating Least Square (ALS) matrix factorization and provide solution to the overfitting problem of the alogrithm. The movie rating data is from [grouplens](https://grouplens.org/datasets/movielens/latest/) (Small: 100,000 ratings and 3,600 tag applications applied to 9,000 movies by 600 users.)

## Data characteristics

A major data characteristic of training a recommendation system is high sparsity. Another important data feature is that the number of movies is around 10 times larger than that of the users. Those features will lead to the overfitting problem discussed later.

![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/user_dis.png)
![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/movie_dis.png)

## Why ALS?

Alternating Least Square (ALS) Matrix Factorization is an intermediary along the pathway of developing recommendation systems and is widely used by many companies. The key characteristic of this approach is a combination of good scalability and predictive accuracy. A detailed description can be found on [Standford CME323](http://stanford.edu/~rezab/classes/cme323/S15/notes/lec14.pdf) and [NETFLIX-recommender systems](https://datajobs.com/data-science-repo/Recommender-Systems-[Netflix].pdf)

# Model training and evaluation

## Data Exploration

The source data is relatively clean. The rating table (useid, movieid, and rating) has no null value and is used for model training. Then the movie table (movieid, name, type) is also useful as it provides more insights into the movie's type and actual content.

As expected, the 'long-tail' characteristic is clearly present in the rating table. The spasticity of the training matrix is as high as **0.983.**

## Training and model selection

The parameters that need tuning are rank (how many latent features for each user and movie), regParam (regulation parameter), and maxIter (max iteration). Grid search and cross-validation are used. Mean squared error (MSE) is used the evaluation metric. Final parameters are determined by choosing model with MSE and then used for training the whole set of data.

## Analysis

To visualize the actual data and the predicted data, my first step is to round the predicted score to 0.5. 

![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/rounded_ratings.png)

When all ratings are plotted against the userid and movieid, the graph is too messy. 

![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/ALL.png)

So, we can look at the MSE for 4 special cases (users rated most and least movies, and movies with most and least users) for a better understanding of the result. In terms of users, the prediction for who rated most movies is better than those who rated least, which is intuitively right. However, in terms of movies, it is surprising to find out that the prediction for the movie rated by most users is far worse than that for movies with only one rating.

| MSE | User  | Movie |
| -- | -- | -- |
| Most | 0.578 | 0.571 |
| Least  | 0.748 | 0.021 |

This is also not due to the effect of averaging. 

![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/most_by_movies.png)
![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/least_by_movies.png)

This is due to the overfitting problem mentioned above. Let's consider what the matrix factorization actually does. It is trying to guess the 'most likely' k-dimension features for each movie (m) and user (n) based on an n by m matrix with labeled ratings. 

![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/Matrix-Factorization.png)

In a 2x2 rating table, ff we set k=1, normally we can have solutions without error. That means we certainly know the taste of each user and the the latent features of each movie. If we lose one rating, to guess the rating by using matrix factorization for this missing value is very risky or actually invalid, because there are infinite combinations of results. Increasing k would lead to a worse scenario.

| rating (1-5) | movie 1 | movie 2 |
| -- | -- | -- |
| user 1 | 3 | 4 |
| user 2 | 1 | 5 |

Based on the analysis above, we can quickly imagine a 1xn or an nx1 table will also cause overfitting. So ideally, we need a reasonable 'activity' of movies and users and also a reasonable 'correlation' among users and movies. 

To decide the threshold, I plot the MSE averaged by the number of ratings for each movie. When the rating number is lower than 50, the prediction error is of high variance. Therefore, ideally, I would recommend 50 to be the minimal number of ratings for both users and movies. So we should encourage users who rated less than 50 movies rating more movies and collect more ratings for movies with less than 50 ratings.

![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/error_by_movie.png)

## Applications

We can provide recommendations for users based on this model and find similar users or movies according to the cosine correlation of the latent features. The following table shows the recommendation result for user (232, 575) using this model. We can say the model gives a reasonable recommendation for users given the type of movies are consistent.

![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/recommended_1.png)

It is also the same procedure to find similar movies for (471). But this model faces the problem that it can not predict if the movie has not been rated before.

![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/recommended.png)

Beyond that, we should also pay special to attention to the users who are not represented well by the model (Also true for movies). 

# Conclusion

Using ALS-based Matrix Factorization, I built a movie recommendation systems. The data displayed high sparsity and an unbalanced ratio between the number users and movies. This a major reason for the overfitting problem. 

Considering the application, I would recommend setting 50 as the threshold for the number of ratings for training better model. It infers that we should actively encourage people to rate movies when they have small amount of ratings or the movie have a small number of ratings. Also, for those less-represented users and movies by the model, companies could also build a separate model for them.
