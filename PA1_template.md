---
title: "Reproducible Research - Project 1"
author: "Carlos Cortez"
date: "January 22, 2019"
output: html_document
---

## Loading and preprocessing the data

1. Load the data


```r
activity <- read.csv("./activity.csv",header = TRUE)
```

2. Process/transform the data (if necessary) into a format suitable for your analysis


```r
activity$day <- weekdays(as.Date(activity$date))
activity$DateTime<- as.POSIXct(activity$date, format="%Y-%m-%d")

# Pulling data without NAs
clean <- activity[!is.na(activity$steps),]
```


## What is mean total number of steps taken per day?

1. Calculate the total number of steps taken per day


```r
# Summarizing total steps per day
sumTable <- aggregate(activity$steps ~ activity$date, FUN=sum)
colnames(sumTable) <- c("Date", "Steps")
```

2. Make a histogram of the total number of steps taken each day


```r
# Creating the historgram of total steps per day
hist(sumTable$Steps, breaks=5, xlab="Steps", main = "Total Steps per Day")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-4-1.png" width="672" />

3. Calculate and report the mean and median of the total number of steps taken per day


```r
# Mean of Steps
as.integer(mean(sumTable$Steps))
```

```
## [1] 10766
```

```r
# Median of Steps
as.integer(median(sumTable$Steps))
```

```
## [1] 10765
```

The average number of steps taken each day was 10766 steps.

The median number of steps taken each day was 10765 steps.

## What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "1") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
# Create average number of steps per interval
StepsPerInterval <- tapply(activity$steps, activity$interval, mean, na.rm = TRUE)

# Create line plot of average number of steps per interval
plot(as.numeric(names(StepsPerInterval)), 
     StepsPerInterval, 
     xlab = "Interval", 
     ylab = "Steps", 
     main = "Average Daily Activity Pattern", 
     type = "l")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-6-1.png" width="672" />

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
# Maximum steps by interval
as.integer(maxInterval <- names(sort(StepsPerInterval, decreasing = TRUE)[1]))
```

```
## [1] 835
```

```r
# Which interval contains the maximum average number of steps
as.integer(maxSteps <- sort(StepsPerInterval, decreasing = TRUE)[1])
```

```
## [1] 206
```

The maximum number of steps for a 5-minute interval was 835 steps.

The 5-minute interval which had the maximum number of steps was the 206 interval.

## Imputing missing values

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
# Number of NAs in original data set
nrow(activity[is.na(activity$steps),])
```

```
## [1] 2304
```

The total number of rows with steps = ‘NA’ is 2304.

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

My strategy is to fill in missing data with the mean number of steps across all days with available data for that particular interval.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.



```r
StepsPerInterval <- tapply(activity$steps, activity$interval, mean, na.rm = TRUE)
# Split activity data by interval
activity.split <- split(activity, activity$interval)
# Fill in missing data for each interval
for(i in 1:length(activity.split)){
    activity.split[[i]]$steps[is.na(activity.split[[i]]$steps)] <- StepsPerInterval[i]
}
activity.imputed <- do.call("rbind", activity.split)
activity.imputed <- activity.imputed[order(activity.imputed$date) ,]
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
StepsPerDay.imputed <- tapply(activity.imputed$steps, activity.imputed$date, sum)
hist(StepsPerDay.imputed, xlab = "Number of Steps", main = "Histogram: Steps per Day (Imputed data)")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-10-1.png" width="672" />


```r
# Mean of steps taken per day
as.integer(MeanPerDay.imputed <- mean(StepsPerDay.imputed, na.rm = TRUE))
```

```
## [1] 10766
```

```r
# Median of steps taken per day
as.integer(MedianPerDay.imputed <- median(StepsPerDay.imputed, na.rm = TRUE))
```

```
## [1] 10766
```

The mean and median number of steps taken per day including imputed data are 10766 and 10766, respectively. The mean remains the same as prior to imputation, while the median value increased slightly.

## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
# Create new category based on the days of the week
activity.imputed$day <- ifelse(weekdays(as.Date(activity.imputed$date)) == "Saturday" | weekdays(as.Date(activity.imputed$date)) == "Sunday", "weekend", "weekday")
```

2. Make a panel plot containing a time series plot (i.e. type = "1") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
# Calculate average steps per interval for weekends
StepsPerInterval.weekend <- tapply(activity.imputed[activity.imputed$day == "weekend" ,]$steps, activity.imputed[activity.imputed$day == "weekend" ,]$interval, mean, na.rm = TRUE)

# Calculate average steps per interval for weekdays
StepsPerInterval.weekday <- tapply(activity.imputed[activity.imputed$day == "weekday" ,]$steps, activity.imputed[activity.imputed$day == "weekday" ,]$interval, mean, na.rm = TRUE)

# Set a 2 panel plot
par(mfrow=c(2,1), mar=c(4,4,2,1))

# Plot weekday activity
plot(as.numeric(names(StepsPerInterval.weekday)), 
     StepsPerInterval.weekday, 
     xlab = "Interval", 
     ylab = "Steps", 
     main = "Activity Pattern (Weekdays)", 
     type = "l")

# Plot weekend activity
plot(as.numeric(names(StepsPerInterval.weekend)), 
     StepsPerInterval.weekend, 
     xlab = "Interval", 
     ylab = "Steps", 
     main = "Activity Pattern (Weekends)", 
     type = "l")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-13-1.png" width="672" />

