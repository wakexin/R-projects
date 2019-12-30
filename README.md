# R-projects
---
title: "Individual Project"
author: "Kexin Wang"
date: "12/1/2019"
output:
  html_document: 
    toc: yes
    toc_float: yes
    theme: lumen
---
# Initialization
```{r}
normalize <- function(x) { 
  return((x - min(x)) / (max(x) - min(x)))
}
```

#0. Clean Data
##First of all, I took a look at the data. There are several variables that are not numerical variables. 
##0.1 For DOB, I turned Date of birth (DOB.x, DOB.y) into age to get valuable insight. To do that, I subtracted the year from date of birth which is in day/month/year formate and divided the number of days to get number of years, which is age.
##0.2 For Stance, I created a new factor "other" to replace player's stance with N/A. A few rows still have n/a values, but predicting those values such as weight and height will create greater errors for the predictions. Therefore, I used function complete.cases to remove reamining rows with NA.
##0.3 For NA values in Reach, I built a model, using height and weight to predict the reach of missing values in reach.
```{r}
fightdata <- read.csv("FightData.csv")
str(fightdata)

fightdata$DOB.x = as.character(fightdata$DOB.x)
fightdata$DOB.x = as.Date(fightdata$DOB.x, format = "%m/%d/%Y")
head(fightdata$DOB.x)
fightdata$age.x = as.numeric(Sys.Date() - fightdata$DOB.x)
fightdata$age.x <- fightdata$age.x/365

fightdata$DOB.y = as.character(fightdata$DOB.y)
fightdata$DOB.y = as.Date(fightdata$DOB.y, format = "%m/%d/%Y")
head(fightdata$DOB.y)
fightdata$age.y = as.numeric(Sys.Date() - fightdata$DOB.y)
fightdata$age.y <- fightdata$age.y/365


fightdata$DOB.x <- NULL
fightdata$DOB.y <- NULL

summary(fightdata$Stance.x)
str(fightdata$Stance.x)
fightdata$Stance.x = as.character(fightdata$Stance.x)
fightdata[fightdata$Stance.x == "Open Stance"|fightdata$Stance.x == "Sideways"|fightdata$Stance.x == "Switch"|is.na(fightdata$Stance.x),]$Stance.x <- "Other"
fightdata = fastDummies::dummy_columns(fightdata, select_columns = "Stance.x")

fightdata$Stance.y = as.character(fightdata$Stance.y)
fightdata[fightdata$Stance.y == "Open Stance"|fightdata$Stance.y == "Sideways"|fightdata$Stance.y == "Switch"|is.na(fightdata$Stance.y),]$Stance.y <- "Other"
fightdata = fastDummies::dummy_columns(fightdata, select_columns = "Stance.y")


fightdata$Stance.x<-NULL
fightdata$Stance.y<-NULL

fightdata$y<- fightdata$result
fightdata$result <- NULL

fightdata2 <- fightdata[complete.cases(fightdata),]
model1 <- lm(Reach.x ~ Height.x + Weight.x, data = fightdata2)
summary(model1)

predicted_Reach_X = predict(model1, newdata = fightdata)
Reach_X_comp = data.frame(predicted_Reach_X, fightdata$Reach.x)


fightdata$Reach.x = ifelse(is.na(fightdata$Reach.x), predicted_Reach_X, fightdata$Reach.x)


model2 = lm(Reach.y ~ Height.y + Weight.y, data = fightdata2)
summary(model2)

predicted_Reach_Y = predict(model2, newdata = fightdata)
Reach_Y_comp = data.frame(predicted_Reach_Y, fightdata$Reach.y)
fightdata$Reach.y = ifelse(is.na(fightdata$Reach.y), predicted_Reach_Y, fightdata$Reach.y)

colMeans(is.na(fightdata))
str(fightdata)
```


##0.4 Now, let's take a look at numerical and int variables. In this part, I handled the '0' value in fightdata by replacing them with the mean value.For rows that has too many NA, I used as.complete function to remove those rows to ensure accuracy of my model later on. 
```{r}

meanSAPm.x <- mean(fightdata[fightdata$SApM.x != 0,]$SApM.x)
meanSAPm.y <- mean(fightdata[fightdata$SApM.y != 0,]$SApM.y)
fightdata$SApM.x <- ifelse(fightdata$SApM.x == 0, meanSAPm.x, fightdata$SApM.x)
fightdata$SApM.y <- ifelse(fightdata$SApM.y == 0, meanSAPm.y, fightdata$SApM.y)


meanSLpM.x <- mean(fightdata[fightdata$SLpM.x != 0,]$SLpM.x)
meanSLpM.y <- mean(fightdata[fightdata$SLpM.y != 0,]$SLpM.y)
fightdata$SLpM.x <- ifelse(fightdata$SLpM.x == 0, meanSLpM.x, fightdata$SLpM.x)
fightdata$SLpM.y <- ifelse(fightdata$SLpM.y == 0, meanSLpM.y, fightdata$SLpM.y)

meanStr..Acc..x <- mean(fightdata[fightdata$Str..Acc..x != 0,]$Str..Acc..x)
meanStr..Acc..y <- mean(fightdata[fightdata$Str..Acc..y != 0,]$Str..Acc..y)
fightdata$Str..Acc..x <- ifelse(fightdata$Str..Acc..x == 0, meanStr..Acc..x, fightdata$Str..Acc..x)
fightdata$Str..Acc..y <- ifelse(fightdata$Str..Acc..y == 0, meanStr..Acc..y, fightdata$Str..Acc..y)

meanStr..Def..x <- mean(fightdata[fightdata$Str..Def..x!= 0,]$Str..Def..x)
meanStr..Def..y <- mean(fightdata[fightdata$Str..Def..y!= 0,]$Str..Def..y)
fightdata$Str..Def..x <- ifelse(fightdata$Str..Def..x  == 0, meanStr..Def..x, fightdata$Str..Def..x)
fightdata$Str..Def..y <- ifelse(fightdata$Str..Def..y  == 0, meanStr..Def..y, fightdata$Str..Def..y)

meanTD.Avg..x <- mean(fightdata[fightdata$TD.Avg..x!= 0,]$TD.Avg..x)
meanTD.Avg..y <- mean(fightdata[fightdata$TD.Avg..y!= 0,]$TD.Avg..y)
fightdata$TD.Avg..x <- ifelse(fightdata$TD.Avg..x  == 0, meanTD.Avg..x, fightdata$TD.Avg..x)
fightdata$TD.Avg..y <- ifelse(fightdata$TD.Avg..y  == 0, meanTD.Avg..y, fightdata$TD.Avg..y)

meanTD.Acc..x <- mean(fightdata[fightdata$TD.Acc..x!= 0,]$TD.Acc..x)
meanTD.Acc..y <- mean(fightdata[fightdata$TD.Acc..y!= 0,]$TD.Acc..y)
fightdata$TD.Acc..x <- ifelse(fightdata$TD.Acc..x  == 0, meanTD.Acc..x, fightdata$TD.Acc..x)
fightdata$TD.Acc..y <- ifelse(fightdata$TD.Acc..y  == 0, meanTD.Acc..y, fightdata$TD.Acc..y)

meanTD.Def..x <- mean(fightdata[fightdata$TD.Def..x != 0,]$TD.Def..x)
meanTD.Def..y <- mean(fightdata[fightdata$TD.Def..y!= 0,]$TD.Def..y)
fightdata$TD.Def..x <- ifelse(fightdata$TD.Def..x == 0, meanTD.Def..x, fightdata$TD.Def..x)
fightdata$TD.Def..y <- ifelse(fightdata$TD.Def..y == 0, meanTD.Def..y, fightdata$TD.Def..y)

meanSub..Avg..x <- mean(fightdata[fightdata$Sub..Avg..x != 0,]$Sub..Avg..x)
meanSub..Avg..y <- mean(fightdata[fightdata$Sub..Avg..y != 0,]$Sub..Avg..y)
fightdata$Sub..Avg..x <- ifelse(fightdata$Sub..Avg..x == 0, meanSub..Avg..x, fightdata$Sub..Avg..x)
fightdata$Sub..Avg..y <- ifelse(fightdata$Sub..Avg..y == 0, meanSub..Avg..y, fightdata$Sub..Avg..y)

colMeans(fightdata == 0)
colMeans(is.na(fightdata))

fightdata = fightdata[complete.cases(fightdata),]

levels(fightdata$y) <- c(0,1)
fightdata$y = as.numeric(fightdata$y)
fightdata <- as.data.frame(lapply(fightdata, normalize))


```

#1.Build three models, I chosed three models: logistic model, knn models and SVM to compare the result of predictions.
##First, we randomize the data, splitting 80% of the data as train data and 20% of the data as test data.
```{r}
data_rand <- fightdata[sample(nrow(fightdata)),]
train <- data_rand[1:ceiling(.8*nrow(fightdata)),]
test <- data_rand[(ceiling(.8*nrow(fightdata)+1)) : nrow(fightdata),]
nrow(train)
nrow(test)
```

## Logistic Model
```{r}
## 1.1 Frist I used all the variables and used logistic model to predict.
logit.model1<- glm(y~ ., data = train,family = "binomial")
summary(logit.model1)

## 1.2 I improved logistic model by using step function to find significant variables and plugged them back in the logisitic model, which results in a higher accuracy rate  and higher kappa value

'step(logit.model1)'

logit.model2 <- glm(y ~ Height.x + Reach.x + SLpM.x + SApM.x + Str..Def..x + 
    TD.Avg..x + TD.Acc..x + TD.Def..x + Height.y + Weight.y + 
    Reach.y + SLpM.y + Str..Acc..y + SApM.y + Str..Def..y + TD.Avg..y + 
    TD.Acc..y + TD.Def..y + Sub..Avg..y + age.x + age.y + Stance.x_Orthodox + 
    Stance.x_Other + Stance.y_Other, family = "binomial", data = train)
summary(logit.model2)

log.predictions1 <- predict(logit.model1, newdata = test, type = "response")
summary(log.predictions1)

log.predictions2 <- predict(logit.model2, newdata = test, type = "response")
summary(log.predictions2)

library(caret)
library(e1071)

log.predictions1 <- ifelse(log.predictions1 >= 0.5, 1, 0)
log.predictions1 <- as.factor(log.predictions1)
levels(log.predictions1) 
test$y <- as.factor(test$y)
levels(test$y) <- c("loss", "win")
levels(log.predictions1) <- c("loss", "win")

confusionMatrix(test$y, log.predictions1)

log.predictions2 <- ifelse(log.predictions2 >= 0.5, 1, 0)
log.predictions2 <- as.factor(log.predictions2)
levels(log.predictions2) 
levels(log.predictions2) <- c("loss", "win")

confusionMatrix(test$y, log.predictions2)


```

### Extract significant variables
```{r}
toselect.x = summary(logit.model1)$coeff[-1,4] < 0.05
relevant.x <- names(toselect.x)[toselect.x == TRUE] 
sig.formula <- as.formula(paste("y ~",paste(relevant.x, collapse= "+")))

```

##knn models
```{r}
##First, we randomize the data, splitting 80% of the data as train data and 20% of the data as test data.
knn_train <- as.data.frame(data_rand[1:ceiling(.8*nrow(data_rand)), -31], drop = FALSE)
knn_test <- as.data.frame(data_rand[ceiling(.8*nrow(data_rand)+1) : nrow(data_rand), -31], drop = FALSE)
train_labels <- data_rand[1:ceiling(.8*nrow(data_rand)),31]
test_labels <- data_rand[ceiling(.8*nrow(data_rand)+1): nrow(data_rand),31]

library(class)
##1.2 I use total observation^(1/2) to get a optimal k
test_pred1 <- knn(train = knn_train, test = knn_test, cl = train_labels, k = 74)

##2.2 I futher improve the knn model by taking another k
library(gmodels)
CrossTable(x= test_labels, y = test_pred1, prop.chisq = FALSE)

test_pred2 <- knn(train = knn_train, test = knn_test, cl = train_labels, k = 50)
CrossTable(x= test_labels, y = test_pred2, prop.chisq = FALSE)

```

## svm
```{r}
library(kernlab)
##First, we randomize the data, splitting 80% of the data as train data and 20% of the data as test data.
svm_train <- as.data.frame(data_rand[1:ceiling(.8*nrow(data_rand)),])
svm_test <- as.data.frame(data_rand[ceiling(.8*nrow(data_rand)+1) : nrow(data_rand), ])

svm_test$y <- as.factor(svm_test$y)
levels(svm_test$y) <- c("loss", "win")

svm.model1 <- ksvm(y ~ Height.x + Reach.x + SLpM.x + SApM.x + Str..Def..x + 
    TD.Avg..x + TD.Acc..x + TD.Def..x + Height.y + Weight.y + 
    Reach.y + SLpM.y + Str..Acc..y + SApM.y + Str..Def..y + TD.Avg..y + 
    TD.Acc..y + TD.Def..y + Sub..Avg..y + age.x + age.y + Stance.x_Orthodox + 
    Stance.x_Other + Stance.y_Other, data = svm_train,
    kernel = "rbfdot")

## The following 3 other models I used to improve the performance of my model
levels(svm_test$y) <- c("loss", "win")

svm.predictions1 <- predict(svm.model1, svm_test)
svm.predictions1 <- ifelse(svm.predictions1>=0.5,1,0)

svm.predictions1 <- as.factor(svm.predictions1)
levels(svm.predictions1) <- c("loss", "win")
agreement <- svm.predictions1 == svm_test$y
table(agreement)

#View(test)
prop.table(table(agreement))

svm_test$y = as.factor(svm_test$y)
confusionMatrix(svm_test$y, svm.predictions1)

svm.model2 <- ksvm(y ~ Height.x + Reach.x + SLpM.x + SApM.x + Str..Def..x + 
    TD.Avg..x + TD.Acc..x + TD.Def..x + Height.y + Weight.y + 
    Reach.y + SLpM.y + Str..Acc..y + SApM.y + Str..Def..y + TD.Avg..y + 
    TD.Acc..y + TD.Def..y + Sub..Avg..y + age.x + age.y + Stance.x_Orthodox + 
    Stance.x_Other + Stance.y_Other, data = svm_train,
    kernel = "polydot")
svm.predictions2 <- predict(svm.model2, svm_test)
svm.predictions2 <- ifelse(svm.predictions2>=0.5,1,0)
svm.predictions2 <- as.factor(svm.predictions2)
levels(svm.predictions2) <- c("loss", "win")
confusionMatrix(svm_test$y, svm.predictions2)

svm.model3 <- ksvm(y ~ ., data = svm_train,
    kernel = "polydot")
svm.predictions3 <- predict(svm.model3, svm_test)
svm.predictions3 <- ifelse(svm.predictions3>=0.5,1,0)
svm.predictions3 <- as.factor(svm.predictions3)
levels(svm.predictions3) <- c("loss", "win")
confusionMatrix(svm_test$y, svm.predictions3)

svm.model4 <- ksvm(sig.formula, data = svm_train,
    kernel = "rbfdot")
svm.predictions4 <- predict(svm.model4, svm_test)
svm.predictions4 <- ifelse(svm.predictions4 >=0.5,1,0)
svm.predictions4 <- as.factor(svm.predictions4)
levels(svm.predictions4) <- c("loss", "win")
confusionMatrix(svm_test$y, svm.predictions4)
```
## As we can see, svm model 1 is the one with highest accuracy

## Final Result Combined
## Combine all the models with highest accuracy
```{r}
levels(svm.predictions1) = c(0,1)
levels(test_pred2) = c(0,1)
levels(log.predictions2) = c(0,1)

svm.predictions1 <- as.numeric(svm.predictions1)-1
log.predictions2 <- as.numeric(log.predictions2)-1
test_pred2 <- as.numeric(test_pred2)-1

combined <- log.predictions2 + svm.predictions1 + test_pred2
combined_prediction <- ifelse(combined>=1.5, 1, 0)
combined_prediction <- as.factor(combined_prediction)

levels(combined_prediction) = c("loss", "win")
confusionMatrix(combined_prediction, test$y)
```
## Kappa value:0, accuaracy: 0.4928, not as high as svm model.

# Insight
## The highest I got is from svm. I learn that data cleaning is the important first step to start building all the models and using height and weight to predict. I also learn that svm is the most accurate model to predict, and combining all the models together does not neccessarily means I will get the highest accuracy result. 



