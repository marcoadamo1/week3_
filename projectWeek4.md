---
title: "Prediction Assignment Writeup"
author: "M. A."
date: "17 October 2018"
output:
  pdf_document: default
  html_document: default
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

# Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

##Dataset

The training data for this project are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

The dataset is available at http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har and it was published with the title "Qualitative activity recognition of weight lifting exercises" (E. Velloso, A. Bulling, H. Gellersen, W. Ugulino, and H. Fuks - Proceedings of the 4th Augmented Human International Conference, 2013, 116--123).


##Getting and Cleaning Data

###Preparing the environment
Loading of the useful libraries
```{r warning=FALSE, message=FALSE}
library(caret)
library(lattice)
library(ggplot2) 
library(randomForest)
library(rpart) 
library(rpart.plot) 
set.seed(123)
```
###Data downloading
Downloading the dataset (if not already locally present).
```{r cache=TRUE}
trainURL = "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
testURL = "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
# download the datasets (results are cached)
train <- read.csv(url(trainURL), na.strings=c("NA","#DIV/0!", ""))
test  <- read.csv(url(testURL), na.strings=c("NA","#DIV/0!", ""))
```
Check the dimensions of the dataset
```{r}
dim(train)
dim(test)
```
###Data cleaning
First we remove the first 7 columns, as not subject of this study. Then we remove the columns with NAs (i.e. missing, null or not present values).
```{r cache=TRUE}
#Keep only columns in which the sum of NAs is null
train <- train[,colSums(is.na(train)) == 0]
test <- test[,colSums(is.na(test)) == 0]

#Remove the first 7 columns
train   <- train[,-c(1:7)]
test <- test[,-c(1:7)]
```

Check that the new dimensions are compatible with the performed operations
```{r}
dim(train)
dim(test)
```

###Exploratory Data Analysis
We first plot the variable "classe" in order to have an idea of how the data is partitioned in the training set.
```{r}
plot(train$classe)
```

###Model development
First we partition the training dataset into 70% training and 30% testing (used to validate the model and assess its performances).
```{r, cache=TRUE}
subset <- createDataPartition(y = train$classe, p=0.70, list=FALSE)
train_train_subset <- train[subset, ] 
train_test_subset <- train[-subset, ]
```
We select a random forest model for the classification as it combines the advantages of decision trees in a more robust algorithm, almost neglecting the overfitting aptitude of decision trees. This allows us to let the algorithm select the relevant parameters on which performing the classification. The performance is tested by classifying the rest of the training set and then the algorithm is applied to the test dataset.

```{r, cache=TRUE}
rf_model <- randomForest(classe ~. , data=train_train_subset, method="class")
```
The so-created rf_model is used to classify the test subset of the training set.
```{r, cache=TRUE}
validation <- predict(rf_model, train_test_subset, type = "class")
```
And, for validation purposes, we plot the confusion matrix and verify that the accuracy (99.4%) and confidence intervals are compatible with our needs. To further improve the model, one can decide to increase the training set to 0.8 when calling createDataPartition, leading to an accuracy of 99.6 %.
```{r, cache=TRUE}
confusionMatrix(validation, train_test_subset$classe)
```


###Model application
The trained model is then applied to the classification of the real test subset.

```{r}
prediction <- predict(rf_model, test, type="class")
# Plot the results
prediction
```
