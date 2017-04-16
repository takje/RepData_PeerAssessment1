---
title: 'Reproducible Research: Peer Assessment 1'
output:
  keep_md: yes
  html_notebook: default
  html_document: default
---
This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

>
* steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)
* date: The date on which the measurement was taken in YYYY-MM-DD format
* interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.


setwd("C:/Users/...")
library(data.table)

initiate R packages
```{r}
library(data.table)
library(ggvis)
```

## Loading and preprocessing the data
```{r}
dt_activity <- fread("activity.csv")
```
## What is mean total number of steps taken per day?
```{r}
sumSteps <- dt_activity[, .(sum(steps)), keyby = date]
```

#### plot total number of steps taken per day
```{r}
sumSteps %>% ggvis(~V1) %>% layer_histograms(fill := "#B74C4C") %>% add_axis("x", title = "Total steps") %>% add_axis("y", title = "Frequency")
```
#### Calculate and report the mean and median of the total number of steps taken per day
```{r}
meanSteps <- sumSteps[, mean(V1, na.rm = TRUE)]
meanSteps
medianSteps <- sumSteps[, median(V1, na.rm = TRUE)]
medianSteps
```
## What is the average daily activity pattern?
```{r}
dt_interval <- dt_activity[, mean(steps, na.rm = TRUE), keyby = interval]
summary(dt_interval)
```

#### Time series plot of the average number of steps taken
```{r}
dt_interval %>% ggvis(~interval, ~V1) %>% layer_lines() %>% add_axis("y", title = "average number of steps")
```

#### Calculate the 5-minute interval that, on average, contains the maximum number of steps
between 08.35 and 08.40 am
```{r}
dt_interval[which.max(dt_interval$V1)]
```
## Imputing missing values
There are 2304 NA's which will be replaced with the overall average steps/interval, that is 37.38
```{r}
summary(dt_activity)
```
imput missing value
```{r}
dt_ReplaceNA <- dt_activity[, lapply(.SD, function(x){ifelse(is.na(x), 37.38, x)})]
summary(dt_ReplaceNA)
```
#### Calculate the total number of steps taken each day after missing values are imputed
```{r}
sumStepsFill <- dt_ReplaceNA[, sum(steps), keyby = date]
```
#### Histogram of the total number of steps taken each day after missing values are imputed. 
```{r}
sumStepsFill %>% ggvis(~V1) %>% layer_histograms(fill := "#56d81a") %>% add_axis("x", title = "Total steps after missing values are imputed") %>% add_axis("y", title = "Frequency")
```
Calculation and the histogram show there is only slight differ in value of steps taken per day after missing value are imputed, however the frequency of the maximum steps/interval has increased.
```{r}
sumStepsFill[, mean(V1)]
sumStepsFill[, median(V1)]
```
## Are there differences in activity patterns between weekdays and weekends?

#### Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.
```{r}
dt_ReplaceNA$date <- as.Date(dt_ReplaceNA$date)
dt_ReplaceNA[, WD := ifelse(weekdays(date) %in% c("Staurday", "Sunday"), "weekend", "weekday")]
dt_ReplaceNA$WD <- as.factor(dt_ReplaceNA$WD)
str(dt_ReplaceNA)
```

```{r}
weekSteps <- dt_ReplaceNA[, mean(steps), keyby = c("WD", "interval")]
```

#### Panel plot containing a time series plot of the 5-minute interval and the average number of steps taken, averaged across all weekday days or weekend days.
```{r}
library(ggplot2)
qplot(interval, V1, data = weekSteps, 
      geom = "line", 
      ylab = "average number of steps",
      main = "Differences in activity patterns between weekdays and weekends",
      facets = WD~.)
```





