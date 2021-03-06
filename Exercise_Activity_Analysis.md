# Exercise Activity Analysis


### Synopsis  

In this report we analyze data from a group of 6 people who exercised and measured their activity using accelerometers on the belt, forearm, arm, and dumbell. A prediction model is built using this data to predict the manner in which the partcipants exercised.

### Data Processing

We downloaded and analyzed the training and test data sets. We used the 
repeated k-fold Cross Validation process with k=10 and number of repeats=3.
TO estimate out of sample error, we looked accuracy, sensitivity, specificity, positive predictive value and negative predictive value in the confusion matrix.



```r
library(caret)
set.seed(32343)
training <- read.csv("pml-training.csv")
dim(training)
```

```
## [1] 19622   160
```

```r
testing <- read.csv("pml-testing.csv")
dim(testing)
```

```
## [1]  20 160
```

```r
summary(training$classe)
```

```
##    A    B    C    D    E 
## 5580 3797 3422 3216 3607
```

```r
#Remove unwanted variables
training1 <- subset(training, select = -c(X,user_name,raw_timestamp_part_1,raw_timestamp_part_2,cvtd_timestamp))
testing1 <- subset(testing, select = -c(X,user_name,raw_timestamp_part_1,raw_timestamp_part_2,cvtd_timestamp))

#Convert to numeric except first (new_window) and last (classe or problem_id)
training1[, 2:154] <- sapply(training1[, 2:154], as.numeric)
testing1[, 2:154] <- sapply(testing1[, 2:154], as.numeric)

#Remove NAs in training set based on NAs in testing set
testing2 <- testing1[,colSums(is.na(testing1)) == 0]
testing3 <- subset(testing2, select = -c(problem_id))
training2 <- subset(training1, select = c(names(testing3)))
training2 <- cbind(training2,classe=training1$classe)

#Preprocessing remove zero variance variables
nzv <- nearZeroVar(training2)
training2 <- training2[, -nzv]
testing2 <- testing2[, -nzv]

#Preprocessing remove highly correlated variables
descrCor <- cor(training2[,1:53])
highlyCorDescr <- findCorrelation(descrCor, cutoff = 0.9)
training2 <- training2[, -highlyCorDescr]
testing2 <- testing2[, -highlyCorDescr]

#Preprocessing center and scale
preProcValues <- preProcess(training2[,1:46], method = c("center", "scale"))
trainTransformed <- predict(preProcValues, training2[,1:46])
training2 <- cbind(trainTransformed,classe=training2$classe)

preProcValues <- preProcess(testing2[,1:46], method = c("center", "scale"))
testTransformed <- predict(preProcValues, testing2[,1:46])
testing2 <- cbind(testTransformed,problem_id=testing2$problem_id)

#Reduce variables to the most important ones
modelrpart <- train(classe~., data=training2, method="rpart")
```

```
## Loading required package: rpart
```

```r
varImp(modelrpart)
```

```
## rpart variable importance
## 
##   only 20 most important variables shown (out of 46)
## 
##                   Overall
## num_window          100.0
## magnet_belt_y        69.3
## total_accel_belt     60.9
## yaw_belt             51.4
## pitch_forearm        45.2
## magnet_dumbbell_y    44.1
## accel_dumbbell_y     42.1
## magnet_dumbbell_z    33.3
## roll_forearm         31.1
## roll_dumbbell        27.4
## pitch_belt           26.0
## magnet_arm_x         24.0
## gyros_belt_z         21.2
## magnet_belt_z        19.9
## magnet_dumbbell_x    16.5
## gyros_belt_y          0.0
## magnet_forearm_y      0.0
## roll_arm              0.0
## accel_arm_y           0.0
## gyros_belt_x          0.0
```

```r
#The below variables being selected were the most important ones
training2 <- subset(training2, select=c(num_window,magnet_belt_y,
                                        total_accel_belt,yaw_belt,
                                        pitch_forearm,magnet_dumbbell_y,
                                        accel_dumbbell_y,magnet_dumbbell_z,
                                        roll_forearm,roll_dumbbell,
                                        pitch_belt,magnet_arm_x,
                                        gyros_belt_z,magnet_belt_z,
                                        magnet_dumbbell_x,classe))
testing2 <- subset(testing2, select=c(num_window,magnet_belt_y,
                                        total_accel_belt,yaw_belt,
                                        pitch_forearm,magnet_dumbbell_y,
                                        accel_dumbbell_y,magnet_dumbbell_z,
                                        roll_forearm,roll_dumbbell,
                                        pitch_belt,magnet_arm_x,
                                        gyros_belt_z,magnet_belt_z,
                                        magnet_dumbbell_x,problem_id))


train_control <- trainControl(method="repeatedcv", number=10, repeats=3)
start.time <- Sys.time()
modelrf <- train(classe~., data=training2, trControl=train_control, 
                 method="rf")
confusionMatrix(predict(modelrf, training2[,1:15]), training2$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 5580    0    0    0    0
##          B    0 3797    0    0    0
##          C    0    0 3422    0    0
##          D    0    0    0 3216    0
##          E    0    0    0    0 3607
## 
## Overall Statistics
##                                 
##                Accuracy : 1     
##                  95% CI : (1, 1)
##     No Information Rate : 0.284 
##     P-Value [Acc > NIR] : <2e-16
##                                 
##                   Kappa : 1     
##  Mcnemar's Test P-Value : NA    
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             1.000    1.000    1.000    1.000    1.000
## Specificity             1.000    1.000    1.000    1.000    1.000
## Pos Pred Value          1.000    1.000    1.000    1.000    1.000
## Neg Pred Value          1.000    1.000    1.000    1.000    1.000
## Prevalence              0.284    0.194    0.174    0.164    0.184
## Detection Rate          0.284    0.194    0.174    0.164    0.184
## Detection Prevalence    0.284    0.194    0.174    0.164    0.184
## Balanced Accuracy       1.000    1.000    1.000    1.000    1.000
```

```r
end.time <- Sys.time()
time.taken <- end.time - start.time
time.taken
```

```
## Time difference of 34.87 mins
```


### Results

The model was applied to the testing set. Plots have been created with some of the most important predictors and we see distinct groupings.



```r
predict(modelrf, testing2[,1:15])
```

```
##  [1] E A A E C E E E A E B E D A E D E B E D
## Levels: A B C D E
```

```r
par(mfrow=c(2,2));
qplot(num_window,magnet_belt_y,colour=classe,data=training2)
```

![plot of chunk unnamed-chunk-2](./Exercise_Activity_Analysis_files/figure-html/unnamed-chunk-21.png) 

```r
qplot(magnet_belt_y,total_accel_belt,colour=classe,data=training2)
```

![plot of chunk unnamed-chunk-2](./Exercise_Activity_Analysis_files/figure-html/unnamed-chunk-22.png) 

```r
qplot(total_accel_belt,yaw_belt,colour=classe,data=training2)
```

![plot of chunk unnamed-chunk-2](./Exercise_Activity_Analysis_files/figure-html/unnamed-chunk-23.png) 

```r
qplot(yaw_belt,pitch_forearm,colour=classe,data=training2)
```

![plot of chunk unnamed-chunk-2](./Exercise_Activity_Analysis_files/figure-html/unnamed-chunk-24.png) 
