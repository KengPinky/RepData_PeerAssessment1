---
title: "Project - 1"
author: "Randall T"
date: "2/16/2021"
output:
  html_document:
  keep_md: true
---



## Introduction

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the “quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The data for this assignment can be downloaded from the course web site:

    Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)

The variables included in this dataset are:

    steps: Number of steps taking in a 5-minute interval (missing values are coded as NA\color{red}{\verb|NA|}NA)
    date: The date on which the measurement was taken in YYYY-MM-DD format
    interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

## Load dataset - data was downloaded prior to loading

```r
library("data.table")
library(ggplot2)
library(tidyverse)
library(knitr)

activitydata <- data.table::fread(input = "activity.csv")
```

## What is mean total number of steps taken per day?

1. Calculate total steps per day
2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day
3. Calculate and report the mean and median of the total number of steps taken per day

```r
#get total
total <- activitydata[, c(lapply(.SD, sum, na.rm = FALSE)), .SDcols = c("steps"), by = .(date)]

#plot
ggplot(total, aes(x = steps)) +
        geom_histogram() +
        labs(title = "Daily Steps", x = "Steps", y = "Frequency")
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## Warning: Removed 8 rows containing non-finite values (stat_bin).
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png)

```r
#mean and median
total[, .(mean_steps = mean(steps, na.rm = TRUE), median_steps = median(steps, na.rm = TRUE))]
```

```
##    mean_steps median_steps
## 1:   10766.19        10765
```

## What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
#create interval
interval <- activitydata[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps"), by = .(interval)]

#plot
ggplot(interval, aes(x = interval, y = steps)) +
        geom_line() +
        labs(title = "Average Steps Daily", x = "Interval", y = "Average Steps per day")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png)

```r
#calc man interval
interval[steps == max(steps), .(maxint = interval)]
```

```
##    maxint
## 1:    835
```

## Imputing missing values

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with 𝙽𝙰s)
2.Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
3.Create a new dataset that is equal to the original dataset but with the missing data filled in.
4.Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
nrow(activitydata[is.na(steps),])
```

```
## [1] 2304
```

```r
# fill in missing values
activitydata[is.na(steps), "steps"] <- activitydata[, c(lapply(.SD, median, na.rm = TRUE)), .SDcols = c("steps")]

#create new dataset
data.table::fwrite(x = activitydata, file = "newdata.csv", quote = FALSE)

#histogram of total with mean and median
total <- activitydata[, c(lapply(.SD, sum)), .SDcols = c("steps"), by = .(date)]

total[, .(mean_steps = mean(steps), median_steps = median(steps))]
```

```
##    mean_steps median_steps
## 1:    9354.23        10395
```

```r
ggplot(total, aes(x = steps)) +
        geom_histogram() +
        labs(title = "Dailt Steps", x = "Steps", y = "Frequency")
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png)

## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
2. Make a panel plot containing a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.



```r
#making new variable
new_activity <- activitydata %>%
        mutate(day_of_week = weekdays(x = date))

#creating variable into factor
new_activity[grepl(pattern = "Monday|Tuesday|Wednesday|Thursday|Friday", x = `day_of_week`), "weekday/weekend"] <- "weekday"

new_activity[grepl(pattern = "Saturday|Sunday", x = `day_of_week`), "weekday/weekend"] <- "weekend"

new_activity[, `weekday/weekend` := as.factor(`weekday/weekend`)]       

#create panel plot                
new_activity[is.na(steps), "steps"] <- new_activity[, c(lapply(.SD, median, na.rm = TRUE)), .SDcols = c("steps")]        
interval <- new_activity[, c(lapply(.SD, mean, na.rm = TRUE)), .SDcols = c("steps"), by = .(interval, `weekday/weekend`)] 

ggplot(interval, aes(x = interval, y = steps, color = `weekday/weekend`)) +
        geom_line() +
        labs(title = "Average Steps by type of day", x = "Interval", y = "# of Steps") +
        facet_wrap(~`weekday/weekend`, ncol = 1, nrow = 2)
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png)

