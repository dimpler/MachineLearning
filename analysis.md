---
title: "Course Project"
author: "Sergey Legotsky"
date: "Monday, August 25, 2014"
output: html_document
---


### Getting data and some exploration

In order to build adequate prediction model, the dataset given was cleaned. Majority of variables removed from initial dataset were taken away because of high portion of NA or blank values. The exact cleaning procedure code is available here: [cleaning.data.Rmd](https://github.com/dimpler/MachineLearning). Cleaning procedure was put sepaprately just in order to get consise data analysis report and leave enough space for main analysis. Briefly, for building prediction model here I used 52 predictors; all integer predictors were converted to class numeric.

Data analysis is performed using caret package. Since a lot of steps in current analysis are random we set the seed.


### Splitting the data

For better evaluation of predicition model we leave part of data for cross validation. After building a model we use part of data that we left here for assessing accuracy of model fit.



```r
## Splitting test dataset into training set and cross validation set
inTrain <- createDataPartition(train$classe, p=.75, list=FALSE)
training <- train[inTrain,]
cv <- train[-inTrain,]
rm(train, inTrain)
```

### Prediction model

To optimize time for running prediction algorithm we compress our training set to get as less variablse as possible. This data compression is done by principle component analysis. Here we calculate the number of components required to capture 95% of variance in training data set. So, we focus on ~95% prediction accuracy of our algorythm which could be very attractive target.


```r
preProcess(training[,-53], method="pca", thresh=.95)
```

```
## 
## Call:
## preProcess.default(x = training[, -53], method = "pca", thresh = 0.95)
## 
## Created from 14718 samples and 52 variables
## Pre-processing: principal component signal extraction, scaled, centered 
## 
## PCA needed 25 components to capture 95 percent of the variance
```

Next, we preprocess our variables in training, cross validation and testing data sets using PCA with first 25 first principal components as it was calculated during previous step. 53rd colum was removed from all datasets as it contain the outcome variable to be predicted.


```r
PCApreProc <- preProcess(training[,-53], method="pca", pcaComp=25)
trainingPC <- predict(PCApreProc, training[,-53])
cvPC <- predict(PCApreProc, cv[,-53])
testPC <- predict(PCApreProc, test[,-53])
rm(PCApreProc)
```

The data are ready to build prediction! Here we try to use random forest for our model. We specify outcome variable to be predicted and PCA preprocessed dataset of predictors, method = "rf" will run random forests algorythm for us, method = "cv" will perform 4-fold cross validation, the rest of the options were set due to optimization of calculations rate.


```r
modelFIT <- train(x=trainingPC, y=training$classe, method="rf", trControl = trainControl(method = "cv", number = 4, allowParallel = TRUE, verboseIter = FALSE))
modelFIT
```

From the modelFIT output we see that our random forest model results in 96.9% accuracy on the training set, on the other way this value can be significantly lower while predicting new data due to overfitting. 
To explore possible features in our data that can prove high potential of current result let's see on the first two principal components plot coloured by outcome variable


```r
plot(trainingPC[,2] ~ trainingPC[,1], 
     type="p", 
     col=training$classe,
     xlab="PC_1",
     ylab="PC_2")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 

Here we see that our observations are divided into 5 distinct groups. From one point of view first two principal components do not separate outcome variable by each of spot. But let's hope that some of the rest principal components in other dimensions do it. Let's check our algorythm on cross validation data set.

### Testing builded model on cross vlidation; buillding predictions

Below you can see confusion matrix  of our model on cross validation data set:


```r
confusionMatrix(predict(modelFIT, cvPC), cv$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1391   13    3    1    0
##          B    0  919    7    0    0
##          C    2   17  829   29    6
##          D    2    0   14  773    6
##          E    0    0    2    1  889
## 
## Overall Statistics
##                                         
##                Accuracy : 0.979         
##                  95% CI : (0.975, 0.983)
##     No Information Rate : 0.284         
##     P-Value [Acc > NIR] : <2e-16        
##                                         
##                   Kappa : 0.973         
##  Mcnemar's Test P-Value : NA            
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             0.997    0.968    0.970    0.961    0.987
## Specificity             0.995    0.998    0.987    0.995    0.999
## Pos Pred Value          0.988    0.992    0.939    0.972    0.997
## Neg Pred Value          0.999    0.992    0.994    0.992    0.997
## Prevalence              0.284    0.194    0.174    0.164    0.184
## Detection Rate          0.284    0.187    0.169    0.158    0.181
## Detection Prevalence    0.287    0.189    0.180    0.162    0.182
## Balanced Accuracy       0.996    0.983    0.978    0.978    0.993
```

Here we see that our model results in very attractive sensitivity (higher than 96% for wrost class B) and specificity (more than 99%).

The following code can be used to predict new values for data in testing set. Unfortunately, real testing on separate test set wasn't performed since classe variable wasn't found in testing data st.


```r
pred.classe <- predict(modelFIT, testPC)
table(pred.classe)
```

```
## pred.classe
## A B C D E 
## 7 8 1 1 3
```


### Conclusion

Our data shows that these types of workouts can be analysed with random forests algorythm with very high accuracy.