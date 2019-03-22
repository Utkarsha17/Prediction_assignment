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
    ## lda 0.6919155 0.6986900 0.7054975 0.7059026 0.7124658 0.7243636    0
    ## knn 0.8886463 0.8926299 0.8988365 0.8974311 0.9015111 0.9046579    0
    ## rf  0.9883552 0.9905334 0.9927246 0.9921383 0.9934450 0.9956300    0
    ## 
    ## Kappa 
    ##          Min.   1st Qu.    Median      Mean   3rd Qu.      Max. NA's
    ## lda 0.6100082 0.6187993 0.6273626 0.6278183 0.6357730 0.6507434    0
    ## knn 0.8590665 0.8641893 0.8718681 0.8702162 0.8754941 0.8794071    0
    ## rf  0.9852646 0.9880236 0.9907969 0.9900545 0.9917096 0.9944736    0

1.  Check the predictions of the models and apply them to the test part
    of the train data set to check its performance

<!-- -->

    predictLDA <- predict(linear, newdata=ttest)
    confMatLDA <- confusionMatrix(predictLDA, ttest$classe)
    confMatLDA

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction    A    B    C    D    E
    ##          A 1358  173  115   66   47
    ##          B   37  714  109   39  201
    ##          C  141  162  652  106   85
    ##          D  135   49  124  711  100
    ##          E    3   41   26   42  649
    ## 
    ## Overall Statistics
    ##                                          
    ##                Accuracy : 0.694          
    ##                  95% CI : (0.682, 0.7057)
    ##     No Information Rate : 0.2845         
    ##     P-Value [Acc > NIR] : < 2.2e-16      
    ##                                          
    ##                   Kappa : 0.6126         
    ##  Mcnemar's Test P-Value : < 2.2e-16      
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: A Class: B Class: C Class: D Class: E
    ## Sensitivity            0.8112   0.6269   0.6355   0.7376   0.5998
    ## Specificity            0.9048   0.9187   0.8983   0.9171   0.9767
    ## Pos Pred Value         0.7720   0.6491   0.5689   0.6354   0.8528
    ## Neg Pred Value         0.9234   0.9112   0.9211   0.9469   0.9155
    ## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
    ## Detection Rate         0.2308   0.1213   0.1108   0.1208   0.1103
    ## Detection Prevalence   0.2989   0.1869   0.1947   0.1901   0.1293
    ## Balanced Accuracy      0.8580   0.7728   0.7669   0.8273   0.7882

    predictKNN <- predict(knn, newdata=ttest)
    confMatKNN <- confusionMatrix(predictKNN, ttest$classe)
    confMatKNN

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction    A    B    C    D    E
    ##          A 1599   72   18   11   22
    ##          B   22  969   35   10   34
    ##          C   20   36  930   64   44
    ##          D   29   36   26  862   35
    ##          E    4   26   17   17  947
    ## 
    ## Overall Statistics
    ##                                           
    ##                Accuracy : 0.9018          
    ##                  95% CI : (0.8939, 0.9093)
    ##     No Information Rate : 0.2845          
    ##     P-Value [Acc > NIR] : < 2.2e-16       
    ##                                           
    ##                   Kappa : 0.8757          
    ##  Mcnemar's Test P-Value : < 2.2e-16       
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: A Class: B Class: C Class: D Class: E
    ## Sensitivity            0.9552   0.8507   0.9064   0.8942   0.8752
    ## Specificity            0.9708   0.9787   0.9662   0.9744   0.9867
    ## Pos Pred Value         0.9286   0.9056   0.8501   0.8725   0.9367
    ## Neg Pred Value         0.9820   0.9647   0.9800   0.9792   0.9723
    ## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
    ## Detection Rate         0.2717   0.1647   0.1580   0.1465   0.1609
    ## Detection Prevalence   0.2926   0.1818   0.1859   0.1679   0.1718
    ## Balanced Accuracy      0.9630   0.9147   0.9363   0.9343   0.9310

    predictRF <- predict(rf, newdata=ttest)
    confMatRF <- confusionMatrix(predictRF, ttest$classe)
    confMatRF

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction    A    B    C    D    E
    ##          A 1672    7    0    0    0
    ##          B    1 1130    0    0    1
    ##          C    0    1 1018   15    0
    ##          D    0    1    8  949    6
    ##          E    1    0    0    0 1075
    ## 
    ## Overall Statistics
    ##                                          
    ##                Accuracy : 0.993          
    ##                  95% CI : (0.9906, 0.995)
    ##     No Information Rate : 0.2845         
    ##     P-Value [Acc > NIR] : < 2.2e-16      
    ##                                          
    ##                   Kappa : 0.9912         
    ##  Mcnemar's Test P-Value : NA             
    ## 
    ## Statistics by Class:
    ## 
    ##                      Class: A Class: B Class: C Class: D Class: E
    ## Sensitivity            0.9988   0.9921   0.9922   0.9844   0.9935
    ## Specificity            0.9983   0.9996   0.9967   0.9970   0.9998
    ## Pos Pred Value         0.9958   0.9982   0.9845   0.9844   0.9991
    ## Neg Pred Value         0.9995   0.9981   0.9984   0.9970   0.9985
    ## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
    ## Detection Rate         0.2841   0.1920   0.1730   0.1613   0.1827
    ## Detection Prevalence   0.2853   0.1924   0.1757   0.1638   0.1828
    ## Balanced Accuracy      0.9986   0.9958   0.9945   0.9907   0.9967

    performance <- matrix(round(c(confMatLDA$overall,confMatKNN$overall,confMatRF$overall),3), ncol=3)
    colnames(performance)<-c('Linear Discrimination Analysis (LDA)', 'K- Nearest Neighbors (KNN)','Random Forest (RF)')
    performance.table <- as.table(performance)
    print(performance.table)

    ##   Linear Discrimination Analysis (LDA) K- Nearest Neighbors (KNN)
    ## A                                0.694                      0.902
    ## B                                0.613                      0.876
    ## C                                0.682                      0.894
    ## D                                0.706                      0.909
    ## E                                0.284                      0.284
    ## F                                0.000                      0.000
    ## G                                0.000                      0.000
    ##   Random Forest (RF)
    ## A              0.993
    ## B              0.991
    ## C              0.991
    ## D              0.995
    ## E              0.284
    ## F              0.000
    ## G

1.  After comparing all models, random forest provides better accuracy.
    In random forest model, higher number of nodes give high accuracy
    results. Hence this model is appropriate for this case.

2.  Now, applying prediction model to testing data.

<!-- -->

    prediction <- predict(rf, test)
    prediction

    ##  [1] B A B A A E D B A A B C B A E E A B B B
    ## Levels: A B C D E
