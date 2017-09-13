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

    ##   mtry       ROC      Sens      Spec      ROCSD     SensSD     SpecSD
    ## 1   10 0.8823769 0.9484178 0.4924079 0.01491354 0.01236245 0.03713084
    ## 2   15 0.8833806 0.9467335 0.5098633 0.01485339 0.01074705 0.03568119
    ## 3   20 0.8831637 0.9453844 0.5180233 0.01388299 0.01118325 0.03285475
    ## 4   25 0.8822446 0.9442028 0.5209404 0.01456596 0.01429300 0.03603955
    ## 5   30 0.8828296 0.9433634 0.5244322 0.01403927 0.01261124 0.04138630

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
    ##     NoRain   2822  414
    ##     Rain      159  430
    ##                                                
    ##                Accuracy : 0.8502               
    ##                  95% CI : (0.8385, 0.8614)     
    ##     No Information Rate : 0.7793               
    ##     P-Value [Acc > NIR] : < 0.00000000000000022
    ##                                                
    ##                   Kappa : 0.5115               
    ##  Mcnemar's Test P-Value : < 0.00000000000000022
    ##                                                
    ##             Sensitivity : 0.9467               
    ##             Specificity : 0.5095               
    ##          Pos Pred Value : 0.8721               
    ##          Neg Pred Value : 0.7301               
    ##              Prevalence : 0.7793               
    ##          Detection Rate : 0.7378               
    ##    Detection Prevalence : 0.8460               
    ##       Balanced Accuracy : 0.7281               
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
  print(test2[j, DATE])
  dateList[[dur]] <- test2[j, DATE]
  SpecificityRainList[[dur]] <- temp$byClass["Specificity"]
  SensitivityNoRainList[[dur]] <- temp$byClass["Sensitivity"]
  j <- j + 40
  dur <- dur + 1
  if(i> length(predRF$PredictedTraget)){stop("End of the length")}
  
}
```

    ## [1] "2007-02-19"
    ## [1] "2007-03-31"
    ## [1] "2007-05-10"
    ## [1] "2007-06-19"
    ## [1] "2007-07-29"
    ## [1] "2007-09-07"
    ## [1] "2007-10-17"
    ## [1] "2007-11-26"
    ## [1] "2008-01-05"
    ## [1] "2008-02-14"
    ## [1] "2008-03-25"
    ## [1] "2008-05-04"
    ## [1] "2008-06-13"
    ## [1] "2008-07-23"
    ## [1] "2008-09-01"
    ## [1] "2008-10-11"
    ## [1] "2008-11-20"
    ## [1] "2008-12-30"
    ## [1] "2009-02-08"
    ## [1] "2009-03-20"
    ## [1] "2009-04-29"
    ## [1] "2009-06-08"
    ## [1] "2009-07-18"
    ## [1] "2009-08-27"
    ## [1] "2009-10-06"
    ## [1] "2009-11-15"
    ## [1] "2009-12-25"
    ## [1] "2010-02-03"
    ## [1] "2010-03-15"
    ## [1] "2010-04-24"
    ## [1] "2010-06-03"
    ## [1] "2010-07-13"
    ## [1] "2010-08-22"
    ## [1] "2010-10-01"
    ## [1] "2010-11-10"
    ## [1] "2010-12-20"
    ## [1] "2011-01-29"
    ## [1] "2011-03-10"
    ## [1] "2011-04-19"
    ## [1] "2011-05-29"
    ## [1] "2011-07-08"
    ## [1] "2011-08-17"
    ## [1] "2011-09-26"
    ## [1] "2011-11-05"
    ## [1] "2011-12-15"
    ## [1] "2012-01-24"
    ## [1] "2012-03-04"
    ## [1] "2012-04-13"
    ## [1] "2012-05-23"
    ## [1] "2012-07-02"
    ## [1] "2012-08-11"
    ## [1] "2012-09-20"
    ## [1] "2012-10-30"
    ## [1] "2012-12-09"
    ## [1] "2013-01-18"
    ## [1] "2013-02-27"
    ## [1] "2013-04-08"
    ## [1] "2013-05-18"
    ## [1] "2013-06-27"
    ## [1] "2013-08-06"
    ## [1] "2013-09-15"
    ## [1] "2013-10-25"
    ## [1] "2013-12-04"
    ## [1] "2014-01-13"
    ## [1] "2014-02-22"
    ## [1] "2014-04-03"
    ## [1] "2014-05-13"
    ## [1] "2014-06-22"
    ## [1] "2014-08-01"
    ## [1] "2014-09-10"
    ## [1] "2014-10-20"
    ## [1] "2014-11-29"
    ## [1] "2015-01-08"
    ## [1] "2015-02-17"
    ## [1] "2015-03-29"
    ## [1] "2015-05-08"
    ## [1] "2015-06-17"
    ## [1] "2015-07-27"
    ## [1] "2015-09-05"
    ## [1] "2015-10-15"
    ## [1] "2015-11-24"
    ## [1] "2016-01-03"
    ## [1] "2016-02-12"
    ## [1] "2016-03-23"
    ## [1] "2016-05-02"
    ## [1] "2016-06-11"
    ## [1] "2016-07-21"
    ## [1] "2016-08-30"
    ## [1] "2016-10-09"
    ## [1] "2016-11-18"
    ## [1] "2016-12-28"
    ## [1] "2017-02-06"
    ## [1] "2017-03-18"
    ## [1] "2017-04-27"
    ## [1] "2017-06-06"
    ## [1] NA

``` r
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
