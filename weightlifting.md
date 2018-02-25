# Weight Lifting Prediction

## Introduction
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website  [here](http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har).

The goal of this project is to predict the manner in which they did the exercise.

## Loading and preprocessing the data
Inspecting the data carefully, we note that there are a few divide by zero errors and plenty of missing data on certain columns; this is due to how the data is set up: some readings were only meaningful over a period of time. Technically speaking, a full datum should encapsulate the entire window (after all it is really the entire range of motion we are really interested in).

We also note that the first few 7 variables are either index variables or time dependent variables. For completeness, it may be necessary to check if the variables have unexpected correlations with time (for instance if the weight lifter compromises on form unknowningly after many repetitions). 

Since that will require extensive feature engineering, we shall ignore all erroneous columns for simplicity. Time dependent variables and index variables will also be excluded from consideration. 

```r
# Loading useful libraries
library(caret)
library(dplyr)
```


```r
# Unzip and read file
trainingSet <- read.csv("pml-training.csv")

# Filter columns with NA
filtered <- sapply(trainingSet, function(x) "#DIV/0!" %in% x || is.na(x) )
mydata <- select( trainingSet[filtered==F], roll_belt:classe)
```

## Prediction Model
From the description of the dataset on the website, it is clear that we require some form of decision tree to classify the datasets. We adopt the more versatile and robust random forests (as compared to just decision trees) to predict the above sample. Random forests create many decision trees and votes to pick the best tree out of all of them. 

Due to how computationally expensive random forests are and how poor my computer is, we will do k-fold cross validation with k=3. This enables out of sample error estimation.


```r
set.seed(123456)
train_control <- trainControl(method="cv", number=3)
modelFit <- train(classe~., data=mydata, method="rf", trControl=train_control)
```


```r
modelFit
```

```
## Random Forest 
## 
## 19622 samples
##    52 predictor
##     5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Cross-Validated (3 fold) 
## Summary of sample sizes: 13080, 13081, 13083 
## Resampling results across tuning parameters:
## 
##   mtry  Accuracy   Kappa    
##    2    0.9932726  0.9914895
##   27    0.9930180  0.9911678
##   52    0.9872084  0.9838184
## 
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was mtry = 2.
```

With an estimated out of sample accuracy of 99.327, not bad at all!

## Predicting the testing set

```r
testingSet <- read.csv("pml-testing.csv")
mydataTest <- select( testingSet[filtered==F], roll_belt:magnet_forearm_z)
predictions <- predict(modelFit, newdata=mydataTest)
```
