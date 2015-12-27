# Module8-Pratical Machine Learning - Project
neocklee  
December 22, 2015  



## Executive Summary
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. 

In this papaer, we perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: [1](http://groupware.les.inf.puc-rio.br/har) (see the section on the Weight Lifting Exercise Dataset). 

The goal of your project is to predict the manner in which they did the exercise. This is the "classe" variable in the training set: 

1. exactly according to the specification (Class A)

2. throwing the elbows to the front (Class B)

3. lifting the dumbbell only halfway (Class C)

4. lowering the dumbbell only halfway (Class D)

5. throwing the hips to the front (Class E).

## Data 

The training data for this project are available here: 
[link](https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv)
The test data are available here: 
[link](https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv)


```r
library(caret)
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
library(ggplot2)
dfTraining <- read.csv(file="pml-training.csv",head=TRUE,sep=",")
dfTraining$classe <- as.factor(dfTraining$classe) # make it as factor variable
```
### Data Preprocessing
The original dataset has 160 variables. Variables that having too many NAs, non-numeric variables will be removed to preseve the most useful predictors for further analysis.

Removal of Non-numeric Variables

```r
set.seed(2525) # Set seed for reproductive analysis
dfTraining <- dfTraining[,-(1:7)] # remove first 7 columns
```

Removal of variables wth missing values

```r
colWithMissingValue <- sapply(dfTraining, function (x) any(is.na(x) | x == ""))
dfTraining <- dfTraining[,names(colWithMissingValue)[!colWithMissingValue]]
```

Split the dataset into a 60% training and 40% probing dataset.

```r
dfInTrain <- createDataPartition(dfTraining$classe, p=0.6, list = FALSE)
train <- dfTraining[dfInTrain,]
probe <- dfTraining[-dfInTrain,]
```
Setting up the parallel clusters for better performance.

```r
require(parallel)
require(doParallel)
cluter <- makeCluster(detectCores() - 1)
registerDoParallel(cluter)
control <- trainControl(classProbs=TRUE,
                     savePredictions=TRUE,
                     allowParallel=TRUE)
```
Calculation of variable importance for regression and classification models

```r
modelFit <- train(classe ~ ., data=train, method='rf')
plot(varImp(modelFit), top = 20)
```

## Train a prediction model


```r
modelFit <- train(classe ~ ., data = train, method="rf",importance=TRUE)
accuracy <- confusionMatrix(probe$classe,  predict(modelFit, probe) )
accuracy
stopCluster(cluter)
```



### References
[1] Ugulino, W.; Cardador, D.; Vega, K.; Velloso, E.; Milidiu, R.; Fuks, H. Wearable Computing: Accelerometers' Data Classification of Body Postures and Movements. Proceedings of 21st Brazilian Symposium on Artificial Intelligence. Advances in Artificial Intelligence - SBIA 2012. In: Lecture Notes in Computer Science. , pp. 52-61. Curitiba, PR: Springer Berlin / Heidelberg, 2012. ISBN 978-3-642-34458-9. DOI: 10.1007/978-3-642-34459-6_6.
Read more: http://groupware.les.inf.puc-rio.br/har#sbia_paper_section#ixzz3vVGaYaCG