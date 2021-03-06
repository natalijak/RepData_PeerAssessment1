---
title: "Reproducable Researches Assignment 1"
author: "Natalijak"
date: "7. Februar 2016"
output: html_document
---
```{r}
library(ggplot2)
```

Code for reading and data processing of the csv file "activity.csv", packed in "repdata-data-activity.zip"
Unordered process steps
* Read data 
* Assign dimension type
* Check data
* Create additional column which stores the month only. This will be used for the charts / when plotting data it supports the readability , e.g which data day intervals belongs to which month
* Exclude the NA values (these will be later included into new data set for the calculation of median/mean)
```{r}
data_activity <- read.csv("activity.csv", colClasses = c("integer", "Date", "factor"))
head(data_activity)
names(data_activity)
summary(data_activity)
data_activity$month <- as.numeric(format(data_activity$date, "%m"))
data_noNA <- na.omit((data_activity))
head(data_noNA)
dim(data_noNA)

# Get list of available days
uniqueDates <- unique(dates)
show(uniqueDates)
# Get available intervals
uniqueIntervals <- unique(data_noNA$interval)
show(uniqueIntervals)
show(data_noNA)

```

#Questions associated to the assignment

##What is the mean and total number of steps taken per day? 
* For this part of the assignment, you can ignore the missing values in the dataset.
* Calculate the total number of steps taken per day

```{r}
stepsSplit <- split(data_noNA$steps, dates$yday)
totalStepsPerDay <- sapply(stepsSplit, sum, na.rm = TRUE)
print(totalStepsPerDay)
```

* Make a histogram of the total number of steps taken each day

```{r}
ggplot(data_noNA, aes(date, steps)) + geom_bar(stat = "identity", colour = "darkgreen", fill = "darkgreen", width = 0.5) + facet_grid(. ~ month, scales = "free") + labs(title = "Total Number of Steps Taken Each Day", x = "Date", y = "Total number of steps")

```

* Mean and Median  of steps taken per day:

```{r}
totSteps <- aggregate(data_noNA$steps, list(Date = data_noNA$date), FUN = "sum")$x
mean(totSteps) # Result Median total number of steps taken per day:
#[1] 10766.19

median(totSteps)
#[1] 10765

```

## What is the average daily activity pattern?
* Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged accross all days (y-axis)

```{r}
avgSteps <- aggregate(data_noNA$steps, list(interval = as.numeric(as.character(data_noNA$interval))), FUN = "mean")
#assigne the column name to the calculated mean, second column
names(avgSteps) [2]<- "meanOfSteps"
show(avgSteps)  

#plot the data
ggplot(avgSteps, aes(interval, meanOfSteps)) + geom_line(color = "darkgreen", size = 0.8) + labs(title = "Time Series Plot of the 5-minute Interval", x = "5-minute intervals", y = "Average Number of Steps Taken")
```


* Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```{r}

avgSteps[avgSteps$meanOfSteps == max(avgSteps$meanOfSteps), ]
#interval   meanOfSteps
 # 835    206.1698

```

##Inputing missing values
* Calculate and report the total number of missing values in the dataset (total number of rows with NAs)

```{r}
total_NA <- sum(is.na(data_activity))
# print(total_NA)
# [1] 2304
```

* Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be  sophisticated. 
For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```{r}
sum(is.na(data_activity))
# [1] 2304 to be manipulated

```

Approach: using the mean for 5-minute interval, replace each NA value in the column = steps.
        
* Create a new dataset that is equal to the original dataset but with the missing data filled in.

```{r}
        data_manipulated <- data_activity 
        for (i in 1:nrow(data_manipulated)) 
        {
                if (is.na(data_manipulated$steps[i])) 
                {
                        data_manipulated$steps[i] <- avgSteps[which(data_manipulated$interval[i] == avgSteps$interval), ]$meanOfSteps
                }
        }
        
        
sum(is.na(data_manipulated))
        # [1] 0 
head(data_manipulated)          
        
```

* Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. 

```{r}
ggplot(data_manipulated, aes(date, steps)) + geom_bar(stat = "identity",
                                                     colour = "darkgreen",
                                                     fill = "darkgreen",
                                                     width = 0.7) + facet_grid(. ~ month, scales = "free") + labs(title = "Total Number of Steps Taken Each Day", x = "Date", y = "Total number of steps")
        
```        
     
* Do these values differ from the estimates fromt he first part of the assignment?
```{r}

newTotSteps <- aggregate(data_manipulated$steps, list(Date = data_manipulated$date), FUN = "sum")$x
new_mean <- mean(newTotSteps)
## [1] 10766.19
new_median <- median(newTotSteps)
## [1] 10766.19

old_mean <- mean(totSteps)
# [1] 10766.19
old_median <- median(totSteps)
print(old_median)
#[1] 10765
# Comparisson old vs new  (after manipulating missing data):
new_mean - old_mean
## [1] 0
new_median - old_median
## [1] 1.188679

```        

# What is the impact of imputing  missing data on the estimates of the total daily number of steps?
Answer: after manipulating the NA rows, the new mean of total steps taken per day is the same as that of the old mean. The new median of total steps taken per day is greater than that of the old median, difference of 1.188679.


# Are there differences in activity patterns between weekdays and weekends?
* Create a new factor variable in the dataset with two levels  "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```{r}
head(data_manipulated)
data_manipulated$weekdays <- factor(format(data_manipulated$date, "%A"))
levels(data_manipulated$weekdays)
levels(data_manipulated$weekdays) <- list(weekday = c("Dienstag", "Donnerstag",
                                                     "Freitag", 
                                                     "Mittwoch", "Montag"),
                                         weekend = c("Saturday", "Sunday"))
levels(data_manipulated$weekdays)
table(data_manipulated$weekdays)


```

* Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```{r}
avgSteps <- aggregate(data_manipulated$steps, 
                      list(interval = as.numeric(as.character(data_manipulated$interval)), 
                           weekdays = data_manipulated$weekdays),
                      FUN = "mean")


# assign the name to the third column
names(avgSteps)[3] <- "meanOfSteps"

show(avgSteps)

library(lattice)
xyplot(avgSteps$meanOfSteps ~ avgSteps$interval | avgSteps$weekdays, 
       layout = c(1, 2), type = "l", 
       xlab = "Interval", ylab = "Number of steps")

```
