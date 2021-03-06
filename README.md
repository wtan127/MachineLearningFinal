# MachineLearningFinal
Machine Learning used to predict how well a dumbbell activity is done

###Executive Summary
10 Participants were asked to use dumbbells in 5 different ways (Class A-E), where Class A means that the exercises were performed correctly, and the other classes represent common mistakes. (source: http://groupware.les.inf.puc-rio.br/har). The dataset was cleaned from 160 variables to 10 based on the following:
- Inconsistencies between training and test set characteristics
- "families" of similar data points, where the plots against the classes looked similar
- To shorten run time on the computer.
After looking at trees and random forests, the random forest was able to corretly predict 20/20 of the test set. Boosting was tried multiple times, but was never able to finish running on my very old computer.

###Data Processing
```{r echo=TRUE}
library(caret)
library(ggplot2)
library(dplyr)
fileURL1 <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
fileURL2 <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
download.file(fileURL1, destfile = "train.csv")
download.file(fileURL2, destfile= "test.csv")
train <- read.csv("train.csv")
test <- read.csv("test.csv")
```
There are 6 people in the train set. There are 20 different time stamps.

###Exploratory Analysis
19622 rows and 160 columns is a lot.
```{r echo=TRUE}
dim(train)
```
looking at str(train), it's clear that there are a bunch of numeric values that are displayed as factors.
```{r echo=TRUE}
qplot(train$classe)
```
looking at classes of both train and test, there are inconsistencies where there are 100 columns of logical values in test that aren't in train. We are only going to use columns where the classes are consistent.
```{r echo=TRUE}
trainclass <- sapply(train, class)
testclass <- sapply(test, class)
table(trainclass)
```
```{r echo=TRUE}
table(testclass)
```
```{r echo=TRUE}
sameclass <- trainclass == testclass
table(sameclass)
```
```{r echo=TRUE}
train2 <- train[ ,sameclass]
test2 <- test[ ,sameclass]
rowhasna <- apply(train2, 1, function(x){any(is.na(x))})
sum(rowhasna)
```
```{r echo=TRUE}
rowhadnatest <- apply(test2, 1, function(x){any(is.na(x))})
sum(rowhadnatest)
```
```{r echo=TRUE}
samename <- names(train2) == names(test2)
sum(samename)
```
Now we can add the classe to the end. For the test set, we assume that the classe is A, which is the null hypothesis.
```{r echo=TRUE}
train2$classe <- train$classe
test2$classe <- as.factor(rep("A", 20))
```

Running the caret packages are too time intensive and keep crashing my computer, so first I ran plots against classe for all the variables and decided on the following visually based on if there are statistical outliers in at least 1 of the classes.
```{r echo=TRUE}
nameindex <- names(train2)
x <- nameindex[c(7,8, 10, 11, 16, 17, 19, 20, 21, 23, 24, 25, 28, 31, 32, 34, 36, 37,42, 46, 47, 48, 53, 56, 57)]
trainsub <- select(train2, c(7,8, 10, 11, 16, 17, 19, 20, 21, 23, 24, 25, 28, 31, 32, 34, 36, 37,42, 46, 47, 48, 53, 56, 57))
testsub <- select(test2, c(7,8, 10, 11, 16, 17, 19, 20, 21, 23, 24, 25, 28, 31, 32, 34, 36, 37,42, 46, 47, 48, 53, 56, 57))
print(x)
```
My computer was still crashing, so I narrowed down the columns further into "families", picking out only the ones that seem to show the most statistical significance.
```{r echo=TRUE}
x2 <- nameindex[c(10, 11, 19, 20, 31, 34, 36, 46, 47, 53, 57)]
trainsub2 <- select(train2, c(10, 11, 19, 20, 31, 34, 36, 46, 47, 53, 57))
testsub2 <- select(test2, c(10, 11, 19, 20, 31, 34, 36, 46, 47, 53, 57))
print(x2)
```

###Method
There are a lot of factors that go into the classe variable. When dealing with the data set, it seems useful to use random forests or boosting methods. I tried to run random forest and gbm boosting with the full dataset, but my very old computer simply cannot handle it. It crashed multiple times times.

Cross Validation: I used 5-fold cross validation, mostly because of the time constraint.
```{r echo=TRUE}
modtree <- train(classe~., method="rpart", data=trainsub, trControl=trainControl(method="cv", number=5))
#print(modtree$finalModel)
predict(modtree, testsub)
```
```{r stillsuperslowrf, echo=TRUE, cache=TRUE}
modrf <- train(classe~., method="rf", data=trainsub2, trControl=trainControl(method="cv", number=5))
predict(modrf, testsub2)
```
gbm is recommended to be used without adjusting the trainControl. I would have wanted to run this, but it is simply not possible given my resources.
modgbm <- train(classe~., method="gbm", data=trainsub2, verbose=FALSE)
predict(modgbm, testsub2)


###Results
The results from the Random Forest was particularly good on the short test data set. It was correct on 20 out of the 20 test samples. The Tree was less accurate, only predicting 8 out of 20, possibly because it was taking the "root" variable, and was thrown off by the null hypothesis of "A" that I put into the classe variable. 

The expected out of sample error will be more significant for more data points. I shorted the data greatly, and was not able to run all of the tests I wanted. I am guessing that boosting would give good results as well, and a combination of boosting and random forest would be net better than either. This is because there were a lot of variables with a wide range of 25%-75%ile that overlap between classes, making them relatively weak predictors. Boosting is specifically created to help in such situations.


###Appendix

```{r echo=TRUE}
table(train$user_name)
```
```{r echo=TRUE}
table(train$cvtd_timestamp)
```

Example of one of the plots I ran to pick the variables
```{r echo=TRUE}
plot(train2$classe, train2[ ,8])
```

variables 38, 39, 40, 51, 52 would need to be log10 I left them out because I did not need them.
