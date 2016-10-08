Instructions
============

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now
possible to collect a large amount of data about personal activity
relatively inexpensively. These type of devices are part of the
quantified self movement – a group of enthusiasts who take measurements
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

Summary
-------

In this project we will build some predictions about the ways in which
participants experiment did the exercise in order to identify the
classification (correctly and incorrectly in 5 different ways).

Global options
--------------

Open libraries.

    library(caret)

    ## Loading required package: lattice

    ## Loading required package: ggplot2

    library(rattle)

    ## Rattle: A free graphical interface for data mining with R.
    ## Version 4.1.0 Copyright (c) 2006-2015 Togaware Pty Ltd.
    ## Type 'rattle()' to shake, rattle, and roll your data.

Setting the working directory and download data.

    if(!file.exists("~/projetos/projetos_R/prediction_assignment_writeup/")){
      dir.create("~/projetos/projetos_R/prediction_assignment_writeup/")
    }
    setwd("~/projetos/projetos_R/prediction_assignment_writeup/")

    fileName <- "pml-training.csv"
    if (!file.exists(fileName)){
      fileURL <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
      download.file(fileURL, fileName, method="curl")
    }

    fileName <- "pml-testing.csv"
    if (!file.exists(fileName)){
      fileURL <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
      download.file(fileURL, fileName, method="curl")
    }

    set.seed(32343)

Openning and cleaning the data
------------------------------

Removed the rows of data which has “NA” and also the variables which are
not required or useful for the predictions from both the training set
and the testing set.

    pml_training <- read.csv("pml-training.csv", header=T, na.strings=c("NA", "#DIV/0!"))
    pml_testing <- read.csv("pml-testing.csv", header=T, na.string=c("NA", "#DIV/0!"))

    pml_training_set <- pml_training[, apply(pml_training, 2, function(x) !any(is.na(x)))]
    pml_testing_set <- pml_testing[, apply(pml_testing, 2, function(x) !any(is.na(x)))]

    pml_training_set <- pml_training_set[,-c(1:7)]
    pml_testing_set <- pml_testing_set[,-c(1:7)]

    dim(pml_training_set)

    ## [1] 19622    53

    dim(pml_testing_set)

    ## [1] 20 53

Create partitions
-----------------

We create the partitions (training and testing) with the
pml\_training\_set data.

    inTrain <- createDataPartition(y=pml_training_set$classe, p=0.75, list=FALSE)
    training <- pml_training_set[inTrain,]
    testing <- pml_training_set[-inTrain,]

Testing models
--------------

We define model\_fit1 using the method "rpart" (Recursive Partitioning
and Regression Trees). The accuracy is around .50.

    model_fit1 <- train(classe ~ .,method="rpart",data=training)

    ## Loading required package: rpart

    fancyRpartPlot(model_fit1$finalModel) 

![](prediction_assignment_writeup_files/figure-markdown_strict/unnamed-chunk-5-1.png)

    pred1 <- predict(model_fit1,newdata=testing)
    cMatrix1 <- confusionMatrix(pred1,testing$classe)
    cMatrix1$table

    ##           Reference
    ## Prediction    A    B    C    D    E
    ##          A 1255  383  382  362  130
    ##          B   25  327   31  134  114
    ##          C  112  239  442  308  236
    ##          D    0    0    0    0    0
    ##          E    3    0    0    0  421

    cMatrix1$overall[1]

    ##  Accuracy 
    ## 0.4985726

The model\_fit2 is defined by the method "rf" (random forest) and that
has a better accuracy than the method "rpart". The accuracy is around
.99.

    model_fit2 <- train(classe ~ .,method="rf",data=training, trControl =trainControl(method = "cv", number=10, allowParallel=TRUE), verbose=FALSE)

    ## Loading required package: randomForest

    ## randomForest 4.6-12

    ## Type rfNews() to see new features/changes/bug fixes.

    ## 
    ## Attaching package: 'randomForest'

    ## The following object is masked from 'package:ggplot2':
    ## 
    ##     margin

    pred2 <- predict(model_fit2,newdata=testing)
    cMatrix2 <- confusionMatrix(pred2,testing$classe)
    cMatrix2$table

    ##           Reference
    ## Prediction    A    B    C    D    E
    ##          A 1393    9    0    0    0
    ##          B    1  936    7    0    0
    ##          C    0    4  845    8    3
    ##          D    0    0    3  796    1
    ##          E    1    0    0    0  897

    cMatrix2$overall[1]

    ##  Accuracy 
    ## 0.9924551

We have created another model using the method "gmm" (Generalized
Boosted Regression Models).

    model_fit3 <- train(classe ~ .,method="gbm",data=training, trControl =trainControl(method = "cv", number=10, allowParallel=TRUE), verbose=FALSE)

    ## Loading required package: gbm

    ## Loading required package: survival

    ## 
    ## Attaching package: 'survival'

    ## The following object is masked from 'package:caret':
    ## 
    ##     cluster

    ## Loading required package: splines

    ## Loading required package: parallel

    ## Loaded gbm 2.1.1

    ## Loading required package: plyr

    pred3 <- predict(model_fit3,newdata=testing)
    cMatrix3 <- confusionMatrix(pred2,testing$classe)
    cMatrix3$table

    ##           Reference
    ## Prediction    A    B    C    D    E
    ##          A 1393    9    0    0    0
    ##          B    1  936    7    0    0
    ##          C    0    4  845    8    3
    ##          D    0    0    3  796    1
    ##          E    1    0    0    0  897

    cMatrix3$overall[1]

    ##  Accuracy 
    ## 0.9924551

Conclusion
----------

From the above results the random forest method provides the best fit
model and it is been considered for testing the test data set.

    predictFinal <- predict(model_fit2, newdata=pml_testing_set)

    predictFinal

    ##  [1] B A B A A E D B A A B C B A E E A B B B
    ## Levels: A B C D E
