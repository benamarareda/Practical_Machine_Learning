---
title: "Machine Learning"
author: "Mohamed Reda"
date: "6/4/2020"
output: 
  html_document: 
    keep_md: yes
---

## Overview
The goal of this study is to examine weight lifting exercices dataset, to investigate how well an activity was performed by the wearer. Six young health participants were asked to perform one set of 10 repetitions of the Unilateral Dumbbell Biceps Curl in five different fashions: identified by the variable "classe". Class A corresponds to the specified execution of the exercise, while the other 4 classes correspond to common mistakes.
At the end of this report, we will have a model to predict from the data collected by different body sensors, how well the activity was performed.  


```r
library(caret)
```
## 1- Loading Data
We first download the data from the url provided and readt it

```r
url_train <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
url_test  <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"

download.file(url_train,"training.csv")
download.file(url_test,"testing.csv")
training <- read.csv("training.csv")
testing  <- read.csv("testing.csv")
```
## 2- Exploratory Data Analysis
Let's examine the overall size of the data  

```r
dim(training)
```

```
## [1] 19622   160
```
We can see that there are a high number of predictors, which will slow down our computing time, so we will try to reduce them.  
First we run a PCA to understand the number of predictors explaining the most of the variance.  

```r
dfpca <- training[, - which(names(training) == "classe")]
mod_pca <- preProcess(dfpca,method="pca",thres=.9)
mod_pca
```

```
## Created from 406 samples and 159 variables
## 
## Pre-processing:
##   - centered (123)
##   - ignored (36)
##   - principal component signal extraction (123)
##   - scaled (123)
## 
## PCA needed 31 components to capture 90 percent of the variance
```
We can see that from 160 predictors that only 31 explains 90 % of the variance. 

## 3- Cleaning the Data  
We start by removing non necessary columns from the training data set.  

### 3-1 Removing columns with all values = NA.  

```r
AllNA <- sapply(training, function(x) mean(is.na(x))) > 0.95
training <- training[,  AllNA == FALSE]
```
### 3-2 Removing columns with zero variances / almost constant value across all the rows

```r
nzf <-nearZeroVar(training)
training <- training[, - nzf]
```
### 3-3 Remove non needed columns (username & timestamps)

```r
training <- training[,-(1:5)]
```
From PCA analysis we can see that there are still 23 variables that have no significant impact on the variability of "Classe". However, we choose to continue working with the current training dataset.  

## 4- Model Training  

### 4-1 Split the training data into training and test sets

```r
set.seed(3410)
inTrain = createDataPartition(training$classe, p = 3/4, list = FALSE)
subTrain <- training[inTrain,]
subTest <-  training[-inTrain,]
```
### 4-2 Train the model using RF & GBM methods

```r
controlRf <- trainControl(method="cv", 5)
mod_rf  <- train(classe ~ .,method = "rf",  data = subTrain, trControl = controlRf, ntree = 250)
controlGBM <- trainControl(method = "repeatedcv", number = 5, repeats = 1)
mod_gbm <- train(classe ~ .,method = "gbm", data = subTrain,  trControl = controlGBM, verbose = FALSE)
```
### 4-3 Predict using previous models

```r
pred_rf <-  predict(mod_rf, subTest)
pred_gbm <- predict(mod_rf, subTest)
```
### 4-4 Confusion Matrix

```r
conf_rf <- confusionMatrix(pred_rf,subTest$classe)
conf_gbm <- confusionMatrix(pred_gbm,subTest$classe)
conf_rf$overall['Accuracy']
```

```
##  Accuracy 
## 0.9989804
```

```r
conf_gbm$overall['Accuracy']
```

```
##  Accuracy 
## 0.9989804
```
Both Random Forest and Generalized Boosted Model have very high accuracy, so we go with random forest model.  

## 5- Prediction
Now we should be able to predict with great accuracy, how well the activity was performed for the testing set

```r
pred_final <- predict(mod_rf,newdata = testing)
pred_final
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```


