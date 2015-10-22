Recommendations

This assign focuses on recommendations using generated data. The following code generates a set of 20 users and a set of 20 user-items:
```{r}
library(recommenderlab, quietly = TRUE)
set.seed(1)
m <- matrix(sample(c(as.numeric(1:5), NA), 400,
                   replace=TRUE, prob=c(0.05, 0.1, 0.15, 0.1, 0.05, 0.55)),
                   ncol=20,
                   dimnames=list(user=paste("u", 1:20, sep = ""),
                                 item=paste("i", 1:20, sep = "")))
head(m)

# The rating matrix can be written externally as a CSV file.
# write.csv(m, "m.csv")
# m.df <- read.csv("m.csv", header = TRUE, row.names = 1)
# head(as.matrix(m.df))

r <- as(m, "realRatingMatrix")
```

1 Create a normalized rating matrix, `r_z`, using $Z$-scores. Inspect the raw rating and Z-normalized ratings using `image()`.
```{r}
## Creating Z-score normalized rating matrix and inspecting with raw ratings.

head(as(r,"dgCMatrix"))
identical(as(r, "matrix"),m)
as(r, "list")
head(as(r, "data.frame"))

#Z-score ratings.
r_z <- normalize(r, method="Z-score")
r_z

#using image() to do visual inspection.

image(r, main = "Raw Ratings")
image(r_z, main = "Normalized Ratings")
```

2 Plot the raw and Z-score normalized rating.
```{r}
## Raw and Normalized Z-score ratings plot.
data(Jester5k)
Jester5k
r <- sample(Jester5k, 1000)
r
rowCounts(r[1,])
as(r[1,], "list")
rowMeans(r[1,])
hist(getRatings(r)) #The figure shoes an interesting distribution where all negative values occur with a almost identical frequency and the positive ratings more frequent with a steady decline towards the rating 10, this distribution can be the result of users with strong rating bias.

hist(getRatings(normalize(r, method="Z-score"))) #The distribution of ratings are closer to a normal distribution after Z-score normalization and have reduced variance.
```

3 Plot the raw user counts and raw item means. Discuss.
```{r}
## Computing raw user counts and raw item means.

hist(rowCounts(r)) #user counts.
hist(colMeans(r)) #raw item means.
```
There are unusually many users with ratings around 30 and users who have rated all jokes,the average ratings per joke look closer to a normal distribution with a mean above 0.


4 Create a recommender based on the popularity of items for the first 15 users using the raw ratings.
```{r}
## Creating a recommender.

recommenderRegistry$get_entries(dataType = "realRatingMatrix")
r <- Recommender(Jester5k[1:15], method = "POPULAR") #recommender for the 1st 15 users using "Popularity of items" 
r
names(getModel(r))
getModel(r)$topN
```

5 Create the top-5 recommendation lists for the last five users.
```{r}
## top-5 recommendation lists for the last five users.
getModel(r)$topN
recom <- predict(r, Jester5k[4996:5000], n=5)
recom
as(recom, "list") #The recommended items can be inspected as a list.
recom <- predict(r, Jester5k[4996:5000], type="ratings") #recommender algorithms can also predict ratings.
recom
as(recom, "matrix")[,1:10]
```

6 Create an evaluation scheme which splits the 20 users into a training set (75%) and a test set (25%). For the test set 5 items will be given to the recommender algorithm and the other items will be held out for computing the error. Use a `goodRating` value of 3.
```{r}
## Evaluating predicted ratings.
e <- evaluationScheme(Jester5k[1:20], method="split", train=0.75,
                      given=5, goodRating=3) #training gets .75 part of the 20 users split, and for test 5 users are "given" and other items held for computing the error.
#good rating value = 3.
e
```

7 Create two recommenders (user-based "UBCF" and item-based collaborative filtering "IBCF"") using the training data. Compute predicted ratings for the known part of the test data (5 items for each user) using the two algorithms.  Calculate the error (RMSE, MSE, and MAE) between the prediction and the unknown part of the test data. Discuss.
```{r}
## Creating two recommenders (user-based "UBCF" and item-based collaborative filtering "IBCF"") using the training data.
r1 <- Recommender(getData(e, "train"), "UBCF") #UBCF recommender.
r1
r2 <- Recommender(getData(e, "train"), "IBCF") #IBCF recommender.
r2
p1 <- predict(r1, getData(e, "known"), type="ratings") #predicted ratings for "known" part of test ratings.
p1
p2 <- predict(r2, getData(e, "known"), type="ratings") #predicted ratings for IBCF.
p2
error <- rbind(calcPredictionAccuracy(p1, getData(e, "unknown")),
               calcPredictionAccuracy(p2, getData(e, "unknown"))) #calculating errors (RMSE, MSE, and MAE) between predicted and unknown parts of test data.
rownames(error) <- c("UBCF","IBCF")
error
```
Here, Item-based collaborative filtering produces a smaller prediction error.

8 create a 5-fold cross validation scheme with the the Given-5 protocol,
i.e., for the test users all but five randomly selected items are withheld for evaluation. Compute the average confusion matrix.
```{r}
## cross validation scheme and average confusion matrix.

#5-fold cross validation scheme with given-5 protocol.

scheme <- evaluationScheme(Jester5k[1:20], method="cross", k=5, given=5,
                           goodRating=5)
scheme

## we will create evaluation scheme to evaluate the recommender method popular. We evaluate top-1,top-3, top-5, top-10, top-15 and top-20 recommendation lists.
results <- evaluate(scheme, method="POPULAR", n=c(1,3,5,10,15,20))
results

#average confusion matrix.

getConfusionMatrix(results) #getConfusionMatrix() will return the confusion matrices for the 5 runs (we used 5-fold cross evaluation) as a list.

#TP, FP, FN and TN are the entries for true positives, false positives, false negatives and true negatives in the confusion matrix. The remaining columns contain precomputed performance measures. The average for all runs can be obtained from the evaluation results directly using avg().

avg(results)
```

9 Plot the ROC curve and the precision-recall plot. Discuss.
```{r}
## Plotting ROC curve and precision-recall plot.
plot(results, annotate=TRUE) #ROC, plots the true positive rate (TPR) against the false positive rate (FPR).
plot(results, "prec/rec", annotate=TRUE) #precision-recall.
```

10 Use the evaluation scheme created above to compare the three recommender algorithms: random items, popular items, and user-based CF based recommendations. Plot the ROC curve and the precision-recall plot with the three methods on each plot. Discuss.
```{r}
## Comparing the three recommender algorithms.
scheme <- evaluationScheme(Jester5k[1:20], method="split", train = .75,
                           k=1, given=5, goodRating=3)
scheme

algorithms <- list(
  "random items" = list(name="RANDOM", param=NULL),
  "popular items" = list(name="POPULAR", param=NULL),
  "user-based CF" = list(name="UBCF", param=list(method="Cosine",
        nn=50, minRating=5)))
results <- evaluate(scheme, algorithms, n=c(1, 3, 5, 10, 15, 20))
results
names(results)
plot(results, annotate=c(1,3), legend="topleft")
plot(results, "prec/rec", annotate=3)
```
For this data set and the given evaluation scheme the user-based and popular item-based CF methods are better than all other methods.
