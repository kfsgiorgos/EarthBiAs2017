Machine Learning Application using `data.table`
================
Giorgos Kaiafas, PhD Researcher <br> mail: <georgios.kaiafas@uni.lu> <br> Kostas Mammas, Statistical Programmer <br> mail: <mammaskon@gmail.com> <br>
EarthBiAs2017, Rhodes Island, Greece

Introduction
------------

### Lecture workflow

The scope of our Machine Learning Application is to illustrate the power of the Random Forest predictive algorithm, which belongs to the ensemble category of algorithms. Our target is to predict whether or not it will rain the next days. So, we have to solve a binary classification problem For illustration purposes we will apply the Random Forest algorithm to a specific station.

First we load the needed packages

``` r
library(tidyverse)
library(data.table)
library(ranger)
library(caret)
```

In case we have not loaded the the feature engineering dataset we have to load it.

``` r
setwd("~/Music/mainEarthBias")
FeaturesDt <- readRDS("/home/giorgosk/Music/mainEarthBias/FeatureEngineeringLastEspana.rds")
options(scipen = 9999L)
```

Pre-processing step
-------------------

At this step we have to divide our dataset into training and test set. The test test must not contain the target value. Also, we have to create a vector which has to contain only the target values in order to quantify how accuarate were the predictions.

``` r
# The column of interest is the "RR_DailyPrecipAmount"
FeaturesDt[RR_DailyPrecipAmount>0, Target := 1]
FeaturesDt[RR_DailyPrecipAmount == 0, Target := 0]
# We exclude all the NA values we have created at the Feature Engineering process
FeaturesDt <- na.omit(FeaturesDt)
# We print the ratio of the classes to see how balanced is the datset
FeaturesDt[, 100 * .N/dim(FeaturesDt)[1], by = Target]
```

    ##    Target       V1
    ## 1:      0 72.89889
    ## 2:      1 27.10111

At this step we divide the dataset to train & test

``` r
# We select a station to subeset the dataset
d1 <- "3967"

station <- copy(FeaturesDt[STAID == d1])
# We exclude the meaningless columns
# "RR_DailyPrecipAmount" has to be excluded because is the column that we created the Binary target
station <- station[, !c("STAID", "MinDate", "MaxDate", "Length", "RR_DailyPrecipAmount"), with=F]

station[, Target:= as.factor(Target)]
levels(station$Target)[levels(station$Target)=="0"] <- "NoRain"
levels(station$Target)[levels(station$Target)=="1"] <- "Rain"
# the 2/3 of the total rows  will be used for the training phase
splitThreshold <- round((2/3)*dim(station)[1], 0)
## Test & Train datasets

train1 <- station[1:splitThreshold]
# The following test dataset does not contain the target variable
test1 <- station[(splitThreshold+1):.N, !("Target"),with=F]
# The following tset dataset contain the target variable
test2 <- station[(splitThreshold+1):.N]
```

Train Random forest using caret & grid serach
---------------------------------------------

``` r
## Only this parameter can be tuned in caret with ranger for Grid Search
rfGrid <-  expand.grid(mtry = c(10, 15, 20, 25, 30))

ctrlRF <- trainControl(method = "repeatedcv", number = 10, savePredictions = TRUE, 
                       classProbs = TRUE, summaryFunction = twoClassSummary, allowParallel = TRUE)


mod_fitRF <- train(Target ~ .,  data=train1,  method="ranger", metric="ROC",
                               trControl = ctrlRF, tuneGrid = rfGrid)
```

    ## Loading required package: e1071

``` r
mod_fitRF$results
```

    ##   mtry       ROC      Sens      Spec      ROCSD      SensSD     SpecSD
    ## 1   10 0.8825841 0.9484252 0.4982592 0.01690794 0.007662614 0.02402107
    ## 2   15 0.8840047 0.9485913 0.5116483 0.01682563 0.011198015 0.03337480
    ## 3   20 0.8829912 0.9484221 0.5198014 0.01769380 0.010619884 0.02762164
    ## 4   25 0.8828362 0.9458946 0.5273528 0.01856481 0.011118100 0.03470464
    ## 5   30 0.8823987 0.9437046 0.5314361 0.01822433 0.011218531 0.03119690

Test Phase
----------

``` r
predRF <- predict(mod_fitRF, newdata=test1, type="prob")
predRF <- as.data.table(predRF)
predRF[NoRain>Rain, PredictedTraget:="0"]
predRF[NoRain<Rain, PredictedTraget:="1"]
predRF[, PredictedTraget:= as.factor(PredictedTraget)]
levels(predRF$PredictedTraget)[levels(predRF$PredictedTraget)=="0"] <- "NoRain"
levels(predRF$PredictedTraget)[levels(predRF$PredictedTraget)=="1"] <- "Rain"
confusionMatrix(data=predRF$PredictedTraget, test2$Target)
```

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction NoRain Rain
    ##     NoRain   2824  408
    ##     Rain      157  436
    ##                                                
    ##                Accuracy : 0.8523               
    ##                  95% CI : (0.8406, 0.8634)     
    ##     No Information Rate : 0.7793               
    ##     P-Value [Acc > NIR] : < 0.00000000000000022
    ##                                                
    ##                   Kappa : 0.5193               
    ##  Mcnemar's Test P-Value : < 0.00000000000000022
    ##                                                
    ##             Sensitivity : 0.9473               
    ##             Specificity : 0.5166               
    ##          Pos Pred Value : 0.8738               
    ##          Neg Pred Value : 0.7352               
    ##              Prevalence : 0.7793               
    ##          Detection Rate : 0.7383               
    ##    Detection Prevalence : 0.8450               
    ##       Balanced Accuracy : 0.7320               
    ##                                                
    ##        'Positive' Class : NoRain               
    ## 

### Diagnostics plot

### Calcualtion

We want to understand how accurate are our TP and TN predictions during time. We are searching an aswer to the question:
`Do we predict better the days that are closer to the traing set?` If we predict worse and worse as time passes it would be better to retrain our model including ore observations. So, we create a plot for Sensitivity & Specificity having as window size 40 days. We calculate the confusion matrix for 40 days rolling slices/every 40 days and get back the Sensitivity & Specificity metrics

``` r
SpecificityRainList <- list()
SensitivityNoRainList <- list()
dateList <- list()
j <- 41
dur <- 1
for(i in c(seq(from = 1, to = length(predRF$PredictedTraget), by = 40))){
  
  temp <- confusionMatrix(data=predRF$PredictedTraget[i:j], test2$Target[i:j])
  dateList[[dur]] <- test2[j, DATE]
  SpecificityRainList[[dur]] <- temp$byClass["Specificity"]
  SensitivityNoRainList[[dur]] <- temp$byClass["Sensitivity"]
  j <- j + 40
  dur <- dur + 1
  if(i> length(predRF$PredictedTraget)){stop("End of the length")}
  
}
dateList[[dur-1]] <- test2[.N, DATE]
datelist1 <- purrr::map(dateList, as.character)

Sens <- unlist(SensitivityNoRainList)
Spec <- unlist(SpecificityRainList)
Datelist <- unlist(datelist1)
```

### Create the plot

We create a data.table containg the Sensitivity & Specificity values across time

``` r
Sens1 <- data.table("a" = Sens, Type = "SensitivityNoRain")
Spec1 <- data.table("a" = Spec, Type = "SpecificityRain")

Spec1[, Time:= Datelist]
Sens1[, Time:= Datelist]
DiagnosticsMeasures <- rbindlist(list(Sens1, Spec1))
DiagnosticsMeasures[, Type:=as.factor(Type)]

ggplot(DiagnosticsMeasures, aes(x = Time, y=a, group=Type)) +
  geom_line(aes(color=Type))+
  geom_smooth(aes(color=Type))+
  theme(legend.position="top")+
  labs(y = "Ratio", title="Rolling Diagnostics")+
  theme(axis.text.x=element_text(angle=90,vjust=0.5))
```

    ## `geom_smooth()` using method = 'loess'

    ## Warning: Removed 3 rows containing non-finite values (stat_smooth).

![](ML_application_using_data_table_files/figure-markdown_github-ascii_identifiers/unnamed-chunk-8-1.png)
