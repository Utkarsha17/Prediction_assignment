Background
----------

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now
possible to collect a large amount of data about personal activity
relatively inexpensively. These type of devices are part of the
quantified self movement - a group of enthusiasts who take measurements
about themselves regularly to improve their health, to find patterns in
their behavior, or because they are tech geeks. One thing that people
regularly do is quantify how much of a particular activity they do, but
they rarely quantify how well they do it. In this project, your goal
will be to use data from accelerometers on the belt, forearm, arm, and
dumbell of 6 participants. They were asked to perform barbell lifts
correctly and incorrectly in 5 different ways. More information is
available from the website here:
<http://groupware.les.inf.puc-rio.br/har> (see the section on the Weight
Lifting Exercise Dataset).

Data
----

The training data for this project are available here:

<https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv>

The test data are available here:

<https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv>

Project Goal
------------

The goal of the project is to predict the manner in which they did the
exercise. This is the "classe" variable in the training set.

Create a report describing how model is built, how cross validation is
used, what sample error does mean, why the choices are made and by using
prediction model to predict 20 different test cases.

Data processing
---------------

    # loading reqyuired libraries
    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    library(caret)

    ## Warning: package 'caret' was built under R version 3.5.2

    ## Loading required package: lattice

    ## Loading required package: ggplot2

    ## Warning: package 'ggplot2' was built under R version 3.5.2

    library(plyr)

    ## -------------------------------------------------------------------------

    ## You have loaded plyr after dplyr - this is likely to cause problems.
    ## If you need functions from both plyr and dplyr, please load plyr first, then dplyr:
    ## library(plyr); library(dplyr)

    ## -------------------------------------------------------------------------

    ## 
    ## Attaching package: 'plyr'

    ## The following objects are masked from 'package:dplyr':
    ## 
    ##     arrange, count, desc, failwith, id, mutate, rename, summarise,
    ##     summarize

    library(gbm)

    ## Warning: package 'gbm' was built under R version 3.5.3

    ## Loaded gbm 2.1.5

    library(randomForest)

    ## Warning: package 'randomForest' was built under R version 3.5.3

    ## randomForest 4.6-14

    ## Type rfNews() to see new features/changes/bug fixes.

    ## 
    ## Attaching package: 'randomForest'

    ## The following object is masked from 'package:ggplot2':
    ## 
    ##     margin

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     combine

    library(rpart)
    library(rpart.plot)

    ## Warning: package 'rpart.plot' was built under R version 3.5.3

    library(RColorBrewer)
    library(rattle)

    ## Warning: package 'rattle' was built under R version 3.5.3

    ## Rattle: A free graphical interface for data science with R.
    ## Version 5.2.0 Copyright (c) 2006-2018 Togaware Pty Ltd.
    ## Type 'rattle()' to shake, rattle, and roll your data.

    ## 
    ## Attaching package: 'rattle'

    ## The following object is masked from 'package:randomForest':
    ## 
    ##     importance

    # Downloading Training data from given URL
    download.file(url = "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", 
                  destfile = "./pml-training.csv", method = "curl")

    train <- read.csv("./pml-training.csv", na.strings=c("NA","#DIV/0!",""))

    # Download the testing data from given URL
    download.file(url = "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv", 
                  destfile = "./pml-testing.csv", method = "curl")

    test <- read.csv("./pml-testing.csv", na.strings=c("NA","#DIV/0!",""))

    #Removing the first 7 rows and all NAs which are related to the time series
    data<-names(test[,colMeans(is.na(test))==0])[8:59]

    #Use the filter applying to clean both data sets and make sure both have same variables for analysis.

    train<- train[,c(data,"classe")]
    test<-test[,c(data,"problem_id")]

    #Separate train dataset to increase performance and accuracy
    inTrain<- createDataPartition(train$classe, p=0.7, list=FALSE)
    trtest<- train[inTrain, ]
    ttest<- train[-inTrain, ]

Analysis and Prediction model
-----------------------------

1.Considering K-fold cross validation for 10 iterations to create
partitions of sample datasets (Validation sets). After fitting a model
on to the training data, its performance is measured against each
validation set and then averaged, gaining a better assessment of how the
model will perform when asked to predict for new observations.

    control <- trainControl(method="cv", number=10)
    metric <- "Accuracy"

1.  Consider linear model at first

<!-- -->

    set.seed(10)
    linear <- train(classe~., data=trtest, method="lda", metric=metric, trControl=control)

1.  k-Nearest Neighbors

<!-- -->

    set.seed(10)
    knn <- train(classe~., data=trtest, method="knn", metric=metric, trControl=control)

1.  Random Forest

<!-- -->

    set.seed(10)
    rf <- train(classe~., data=trtest, method="rf", metric=metric, trControl=control)

1.  Now checking performance of above all models

<!-- -->

    results <- resamples(list(lda=linear, knn=knn, rf=rf))
    summary(results)

    ## 
    ## Call:
    ## summary.resamples(object = results)
    ## 
    ## Models: lda, knn, rf 
    ## Number of resamples: 10 
    ## 
    ## Accuracy 
    ##          Min.   1st Qu.    Median      Mean   3rd Qu.      Max. NA's
    ## lda 0.6885007 0.6959765 0.7024763 0.7027736 0.7068558 0.7219796    0
    ## knn 0.8915575 0.8931366 0.8948328 0.8969946 0.8993814 0.9089585    0
    ## rf  0.9890750 0.9919898 0.9930808 0.9929383 0.9939956 0.9963610    0
    ## 
    ## Kappa 
    ##          Min.   1st Qu.    Median      Mean   3rd Qu.      Max. NA's
    ## lda 0.6058573 0.6157191 0.6241689 0.6240005 0.6290342 0.6487601    0
    ## knn 0.8628974 0.8645274 0.8669405 0.8696579 0.8727412 0.8847291    0
    ## rf  0.9861791 0.9898681 0.9912471 0.9910675 0.9924072 0.9953971    0

1.  Check the predictions of the models and apply them to the test part
    of the train data set to check its performance

<!-- -->

    predictLDA <- predict(linear, newdata=ttest)
    confMatLDA <- confusionMatrix(predictLDA, ttest$classe)

    predictKNN <- predict(knn, newdata=ttest)
    confMatKNN <- confusionMatrix(predictKNN, ttest$classe)


    predictRF <- predict(rf, newdata=ttest)
    confMatRF <- confusionMatrix(predictRF, ttest$classe)

    performance <- matrix(round(c(confMatLDA$overall,confMatKNN$overall,confMatRF$overall),3), ncol=3)
    colnames(performance)<-c('Linear Discrimination Analysis (LDA)', 'K- Nearest Neighbors (KNN)','Random Forest (RF)')
    performance.table <- as.table(performance)
    print(performance.table)

    ##   Linear Discrimination Analysis (LDA) K- Nearest Neighbors (KNN)
    ## A                                0.698                      0.911
    ## B                                0.618                      0.887
    ## C                                0.686                      0.903
    ## D                                0.710                      0.918
    ## E                                0.284                      0.284
    ## F                                0.000                      0.000
    ## G                                0.000                      0.000
    ##   Random Forest (RF)
    ## A              0.994
    ## B              0.992
    ## C              0.992
    ## D              0.996
    ## E              0.284
    ## F              0.000
    ## G

1.  After comparing all models, random forest provides better accuracy.
    In random forest model, higher number of nodes give high accuracy
    results. Hence this model is appropriate for this case.
