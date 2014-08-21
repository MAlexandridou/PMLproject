Building a Prediction Model to Predict Activity Quality 
========================================================
R Packages in need
------------------

```r
library(caret)
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
library(ggplot2)
library(randomForest)
```

```
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
```

```r
library(e1071)
```

Data
----
We collected our data from https://class.coursera.org/predmachlearn-004/human_grading/view/courses/972149/assessments/4/submissions and downloaded them from the following 2 links:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

We will focus entirely on training set(i will refer to it as data) and ignore testing set till the very end.

At first look, the data includes 160 variables , in which only 53 are considered important features.
The rest were either mostly empty, NA or irrelevant as they indicate the time, the name of the person or just a row-counting number.


```r
data<-read.csv("pml-training.csv",colClasses=c(
  rep("NULL",7) ,rep(NA,4), ##11
  rep("NULL",25),rep(NA,13),##49
  rep("NULL",10),rep(NA,9),##68
  rep("NULL",15),rep(NA,3),##86
  rep("NULL",15),NA,##102
  rep("NULL",10),rep(NA,12),##124
  rep("NULL",15),NA,##140
  rep("NULL",10),rep(NA,10)##160
  ))
testdata<-read.csv("pml-testing.csv",colClasses=c(
  rep("NULL",7) ,rep(NA,4), ##11
  rep("NULL",25),rep(NA,13),##49
  rep("NULL",10),rep(NA,9),##68
  rep("NULL",15),rep(NA,3),##86
  rep("NULL",15),NA,##102
  rep("NULL",10),rep(NA,12),##124
  rep("NULL",15),NA,##140
  rep("NULL",10),rep(NA,10)##160
))
```

Creating the Prediction Model
-----------------------------
I consider the sample has medium size, therefore i subsplit it into 60% training set and 40% testing set.

I use seeds so i make sure my research is reproducible.


```r
set.seed(13234)
inTrain=createDataPartition(data$classe,p=0.6,list=F)
training=data[inTrain,]
testing=data[-inTrain,]
```

From all the prediction models, i have chosen random Forest. It has the best accuracy from the models, i have
tested so far, and since our data does not have large sample size , speed is not a hindrance.


```r
modFit<-randomForest(classe~.,data=training)
```

Here we take a look at our model.


```r
modFit
```

```
## 
## Call:
##  randomForest(formula = classe ~ ., data = training) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 7
## 
##         OOB estimate of  error rate: 0.65%
## Confusion matrix:
##      A    B    C    D    E class.error
## A 3342    5    1    0    0    0.001792
## B   15 2258    6    0    0    0.009215
## C    0   16 2035    3    0    0.009250
## D    0    0   23 1905    2    0.012953
## E    0    0    1    4 2160    0.002309
```

To check the accuracy we take the summation of the numerical values on the diagonal over the total
amount of numerical values of our data. OR we just subract the "OOB estimate of error rate from 1".
In other words:

```r
DiagonicalValues<-3342+2258+2035+1905+2160
TrainingAccuracy<-DiagonicalValues/(DiagonicalValues+15+5+16+1+6+23+1+3+4+2)
TrainingAccuracy
```

```
## [1] 0.9935
```

```r
1-0.65*0.01
```

```
## [1] 0.9935
```

Results on Training Set
--------
We use the following commands to check our Prediction Model on testing set and take a look at "new data"'s accuracy .


```r
pred<-predict(modFit,testing)
confusionMatrix(pred,testing$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 2230   10    0    0    0
##          B    0 1505   22    0    0
##          C    1    3 1344   21    2
##          D    0    0    2 1265    2
##          E    1    0    0    0 1438
## 
## Overall Statistics
##                                        
##                Accuracy : 0.992        
##                  95% CI : (0.99, 0.994)
##     No Information Rate : 0.284        
##     P-Value [Acc > NIR] : <2e-16       
##                                        
##                   Kappa : 0.99         
##  Mcnemar's Test P-Value : NA           
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             0.999    0.991    0.982    0.984    0.997
## Specificity             0.998    0.997    0.996    0.999    1.000
## Pos Pred Value          0.996    0.986    0.980    0.997    0.999
## Neg Pred Value          1.000    0.998    0.996    0.997    0.999
## Prevalence              0.284    0.193    0.174    0.164    0.184
## Detection Rate          0.284    0.192    0.171    0.161    0.183
## Detection Prevalence    0.285    0.195    0.175    0.162    0.183
## Balanced Accuracy       0.999    0.994    0.989    0.992    0.999
```

Accuracy: 0.992 is a nice one and also our Out of Sample Error is 0.8%. Therefore,since we have only 0.8% error rate for our predictions, we can now test our model to the real testing set.
Results on Real Training Set
----------------------------


```r
answers<-predict(modFit,testdata)
answers<-as.character(answers)
answers
```

```
##  [1] "B" "A" "B" "A" "A" "E" "D" "B" "A" "A" "B" "C" "B" "A" "E" "E" "A"
## [18] "B" "B" "B"
```

