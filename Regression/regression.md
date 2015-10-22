  Regression

Stamey et al. examined the correlation between the level of prostate-specific antigen and a number of clinical measures in men who were about to receive a radical prostatectomy. The input variables are log cancer volume (`lcavol`), log prostate weight (`lweight`), `age`, log of the amount of benign prostatic hyperplasia (`lbph`), seminal vesicle invasion (`svi`), log of capsular penetration (`lcp`), Gleason score (`gleason`), and percent of Gleason scores 4 or 5 (`pgg45`). The output (response) variable is the log of prostate-specific antigen (`lpsa`). The `train` logical variable indicates whether the observation is part of the training dataset (`train = TRUE`) or the test dataset (`train = FALSE`).

The `prostate` data can be found in the `ElemStatLearn` R package.

1 Read the data into R and compute the summary statistics on both the training and test datasets. Comment on any special features and compare the summaries for the two datasets.  

```{r}
library(ElemStatLearn)
data(prostate)
attach(prostate)
head(prostate)

## Dividing parent dataset into training and test dataset.

trains<-subset(prostate[,], (train=="TRUE")) #for training dataset, train column ==True
tests<-subset(prostate[,], train=="FALSE") #for test dataset, train column==False
#generate summary statistics
summary(trains) 
summary(tests)

### The number of observations in training dataset is 67 and test dataset contains 30. The age group is between 41 and 79 in training whereas it is 43 to 70 in test set. There is significant difference in response variable lpsa with minimum value of -0.43 in training set and it is 0.76 in test dataset. lpsa has -ve min value in training but not in test set. 

#overall no noticeable abnormal behaviour can be determined from summaries of training and test dataset.
```

2 Compute the correlation matrix among the input and the response variables on the training dataset. Plot the corresponding scatterplot matrix and comment on the relationships among the variables.

```{r}
## Correlation matrix
#to remove NA value from the matrix(delete Train Column)
trains$train=NULL     

cor(trains) 
#scatterplot
pairs(~ lpsa + lcavol + lweight + age + lbph + svi + lcp + gleason + pgg45,data=trains)

#comments The response variable "lpsa" has strong positive correlation with "lcavol" among all the input variables followed by "svi" and "lweight". A strong positive correlation exists when cor value is closer to (~) +1.00, while a perfect correlation is when the value is 1.00 as against negative correlation when value is -1.00 and no or insignificant correlation exists when the value is far from these two extreme numbers (+-1.00)
```

3 Fit the linear regression model on the training dataset with `lpsa` as the response to all input variables. Test that the regression coefficients are 0. Discuss in terms of the $p$-values for the individual input variables.

```{r}
## Linear regression model
prostate.lm<-lm(lpsa~lcavol + lweight + age + lbph + svi + lcp + gleason + pgg45, data=trains)
(X <- model.matrix(prostate.lm))
(prostate.sm <- summary(prostate.lm)) #summary of the model.
(coefs <- coef(prostate.lm)) #coeffeciant value of each variable.
p = prostate$rank 
n = prostate$rank + prostate$df

## Comments: We can see directly the result of the test of whether any of the predictors have significance in the model. In other words, whether beta_1 = beta_2 = beta_3 = beta_4 = beta_5 = beta_6 = beta_7 = beta_8 =0. Since the p-value, 0.00079, is so small, this null hypothesis is rejected.
```

4 Compute the `mean squared error` evaluation metric on the test dataset and compare it to the corresponding metric on the training dataset. Discuss.

```{r}
## Mean square error
prostate.sm$sigma^2  #MSE for Training dataset
(prostate.pr <- predict(prostate.lm, tests, se.fit=T)) #MSE for Test dataset
sum((tests$lpsa - prostate.pr$fit)^2)/length(train=="FALSE")
#Mean squared error is MeanSq = SumSq/DF (sum of square is Sum of squares for the regression model, Model, the error term, Residual, and the total, Total.) (DF is degree of freedom for each term).

# mean squared error of test data is smaller than training dataset.
```

5 Perform residual regression diagnostics on the model in 3 to determine if deficiencies are present in the model. Discuss.

```{r}
## residual regression diagnostics on linear model
plot(prostate.lm$fit, prostate.lm$res, xlab="Predicted lpsa", ylab="Residual lpsa")
abline(h=0) 
jackres<- rstudent(prostate.lm)
plot(jackres, ylab="Jacknife residuals",ylim=c(-4,4)) #Jacknifed residual
abline(h=0)
max(abs(jackres)) #largest value for jackres
qt(.025/67,58) #two tailed test, 0.05 significance level, 67=no. of data in training dataset, 58=n-p-1, p-1=degree of freedom.
abline(h=c(-3.56,3.56),lty=2)
##Comment- None of the jacknifed residuals exceed these bounds, so there are no significant outliers (by this test)
```

6 Perform a subet selection of the input variables in the training dataset using the R function `step`. What is your final model? Discuss the proposed final (reduced) model.

```{r}
## Put your R code here.
Null=lm(lpsa~1, data=trains) #Efficient forward selection method used for searching the input variables that fit the model most appropriately and for this we have to specify the starting model and range of the model which we want ot examine in the search.
Full=lm(lpsa~., data=trains)

#We can perform forward selection using the command:
step(Null,scope=list(lower=Null, upper=Full),direction="forward")

#To generate the reduced model and summary
prostate.lmr <- lm(lpsa~lcavol +  lweight + svi + lbph, data=trains)
(prostate.smr<-summary(prostate.lmr))
##Comments: According to this procedure, the best model is the one that includes the variables lcavol, lweight, svi and lbph.
```


7 For the reduced model compute the `mean squared error` evaluation metric on the test dataset and compare it to the corresponding metric on the training dataset. Discuss.

```{r}
## Put your R code here.
(prostate.prr <- predict(prostate.lmr, tests, se.fit=T))
prostate.smr$sigma^2
sum((tests$lpsa - prostate.prr$fit)^2)/length(train=="FALSE")
#comment: Mean squared error is MeanSq = SumSq/DF (sum of square is Sum of squares for the regression model, Model, the error term, Residual, and the total, Total.) (DF is degree of freedom for each term). Square root of MSE gives Root MSE which is estimate of the standard deviation of the error distribution.

# mean squared error of test data is smaller than training dataset.
```

8. Perform residual regression diagnostics on the model in 6 to determine if deficiencies are present in the model. Is the final model reasonable? Discuss.

```{r}
##put your R code here
plot(prostate.lmr$fit, prostate.lmr$res, xlab="Predicted lpsa", ylab="Reduced lpsa")
abline(h=0) 
jack<- rstudent(prostate.lmr)
plot(jack, ylab="Final lpsa",ylim=c(-4,4)) #to identify outliers and obtain final model
abline(h=0)
max(abs(jack)) #(This is the largest jack residual)
qt(.025/67,62) #two tailed test for confidence interval 0.05 where n=67(total no. data), n-p-1=62, p=no. of parameters/predictors in the model
abline(h=c(-3.54,3.54),lty=2)
##Comment: Since no outliers are found outside the bound, The final model is reasonable.
```
