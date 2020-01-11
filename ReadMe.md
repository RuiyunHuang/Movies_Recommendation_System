This is a work based on spark, please find code and results [here](https://databricks-prod-cloudfront.cloud.databricks.com/public/4027ec902e239c93eaaa8714f173bcfc/7965635341886967/262132380329106/5164190521743747/latest.html)

**1. Motivation:**

Recommender systems are widely used today by big companies such as Amazon, Youtube and Netflix for their internet products. A good recommendation system is crucial for their business values. In this work, ALS is used to build a recommendation system based on data from [grouplens](https://grouplens.org/datasets/movielens/latest/) (Small: 100,000 ratings and 3,600 tag applications applied to 9,000 movies by 600 users. Last updated 9/2018.) 

**Data characteristics: **

The major data characteristics of training a recommendation system is the high sparcity. Just consider how many rating you have given to all the products you have ever purchased. One thing to point out is that it does not indicate all of products are barely or never be rated. However, the general case is a small amount of products are heavily rated. Also, a small amount of users gives a major contribution to the total ratings for all products.

The other data feature in current work is that the movie number is around 10 times larger than users.

![image](https://github.com/RuiyunHuang/Movies_Recommendation_System/blob/master/images/user_dis.png)

**Why ALS?: **

Alternating Least Square (ALS) Matrix Factorization is a an intermediate technique along the pathway of developing recommendation system and widely used by many companies. The key feature of this approach is a combination of both good scalability and predictive accuracy. A detailed description can be found on [Standford CME323](http://stanford.edu/~rezab/classes/cme323/S15/notes/lec14.pdf) and [NETFLIX-recommender systems](https://datajobs.com/data-science-repo/Recommender-Systems-[Netflix].pdf)

**2. Implementation:**

**Step.1 OLAP**

The source data is found to be relatively clean. Three data tables are found complete with the only table (links.csv) has null values while this table is also currently not related to the major work here. We will focus on the ratings.csv table since it contains the most important information for data training, and then also the movies.csv for providing more persepective on the movies content for users.

As expected both extreme cases for users and movies are observed as mentioned above. The 'long-tail' feature is clearly observed and the spasicity of the user vs. item matrix is as high as 0.983. 

**Step.2 Data training and model selection**

The parameters need be tuning are rank(how many latent features for each user and movie), regParam(regulation parameter), and maxIter(max iteration).  Grid search and cross validation is used. Final parameters are determined. And then used for training the whole set of data.

**Step.3 Analysis**

For a comparison between the actual and predicted data, the first step is to round the predicted score to 0.5. Then as all rating are plotted against userId and movie Id, the information is too messy. Here, we look at 4 special cases (Most least frequent users and movies) for a better sense on the result. 

On the user part, the prediction for most frequent user is better than least frequent user which is intutively right. However, on the movie part, it is suprising to find out that the prediction for movies with most number of ratings is far less good as those with only one rating. This is also not due to average effect as given by the figures. My guess is that since the number of movies rated only once is large in this data set, if we think of them as a category, the fitting might takes more 'care' to them. This is not the case for users. It might related to the ratio of total movies and users as mentioned in data characteristics part. 

To further explore the result, I plot the average absolute error by movieId and number of ratings. As already reflected by the observation above, when averaged by number of ratings, the abs error is lower while the error converges as number of ratings gets to around 25. We can see although the fewer-rated movie may be fitted better, but the penalty is it's more unstable. So ideally, I would recommend 50 as the minimal number of rating for movie to get fair result. 

**Step.4 Applications**

Based on the result, we can do recommendation for users. Also, find similar user or movies according to the cosine correlation of their latent features.

Beyond that, we should also take care of those users who are not reprensent well by the model(Also true for movies). 

**3. Conclusion**

Using ALS based Matrix Factorization, I explored and built a movie recommendation systems. The data features are high sparsicity and unbalanced number ratio between users and movies. The latter might be a reason for unusually result stated above. 

Considering the application, I would recommend set 50 as threshhold for number of rating for a good prediction on movie. It means we should actively encourage people rating those movies with number of ratings lower than this value. Also, for those less-represented users and movies by the model, companies may want to take actions to acquire more data or at least split them out to build another model for them. 
