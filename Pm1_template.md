=================================================

Setup
-----

### Load packages

    library(ggplot2)

    ## Warning: package 'ggplot2' was built under R version 3.3.2

    library(dplyr)
    library(statsr)

### Loading and preprocessing the data

    setwd("~/R/reproducibleResearch/courseProject1")
    dataset <- read.csv("activity.csv")

Exploratory data analysis
-------------------------

### Question 1: What is average number of steps taken per day?

For this part of the assignment, I am ignoring the missing values in the
dataset.I will calculate the number of steps taken per day and report
the results in histogram.

    dailySteps <- dataset %>% group_by(date) %>% summarize(stepsPerDay = sum(steps))
    ggplot(data = dailySteps, aes(x = stepsPerDay)) + geom_histogram() + labs(x = "Number of steps per day") 

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

    ## Warning: Removed 8 rows containing non-finite values (stat_bin).

![](Pm1_template_files/figure-markdown_strict/calculate%20daily%20number%20of%20steps-1.png)

Next, I will calculate and report the mean and median of the total
number of steps taken per day. What is the average daily activity
pattern?

    medianSteps <- median(dailySteps$stepsPerDay, na.rm = TRUE)
    medianSteps

    ## [1] 10765

    meanSteps <- mean(dailySteps$stepsPerDay, na.rm = TRUE)
    meanSteps

    ## [1] 10766.19

The median number of steps per day is 10765 . The mean number of steps
is 10766 .

### Question 2: What is the average number of steps per interval?

I will make a time series plot of the 5-minute interval (x-axis) and the
average number of steps taken, averaged across all days (y-axis).

    intervalSteps <- dataset %>% group_by(interval) %>% summarize(averageSteps = mean(na.omit(steps)))
    ggplot(data = intervalSteps, aes(x = interval, y = averageSteps)) + geom_point(size = 0, shape = 16) + labs(x = "Interval") + geom_line(color = "red")

![](Pm1_template_files/figure-markdown_strict/plot%20number%20of%20steps%20per%20interval-1.png)

Which 5-minute interval, on average across all the days in the dataset,
contains the maximum number of steps?

    maxSteps <- which.max(intervalSteps$averageSteps)
    intervalSteps[maxSteps,]

    ## # A tibble: 1 × 2
    ##   interval averageSteps
    ##      <int>        <dbl>
    ## 1      835     206.1698

Interval \# 104/288 (starting at 835) has the maximum average number of
steps (206.1698).

### Imputing missing values

There are a number of days/intervals where there are missing values
(coded as NA). The presence of missing days may introduce bias into some
calculations or summaries of the data. I will calculate and report the
total number of missing values in the dataset (i.e. the total number of
rows with NAs).

    rowsWithNA <- sum(is.na(dataset$steps))
    rowsWithNA

    ## [1] 2304

There are 2304 missing values for steps.

I will use the intervall mean to fill in missing values in the original
dataset. This new dataset will be called newDataset.

    newDataset <- dataset %>% group_by(interval) %>% mutate(intervalMeanSteps = mean(na.omit(steps)))
    newDataset$steps[is.na(newDataset$steps)] <- newDataset$intervalMeanSteps[is.na(newDataset$steps)]

Now I will make a histogram of the total number of steps taken each day
and Calculate and report the mean and median total number of steps taken
per day in order to assess if these values differ from the estimates
from the first part of the assignment. What is the impact of imputing
missing data on the estimates of the total daily number of steps?

    dailySteps2 <- newDataset %>% group_by(date) %>% summarize(stepsPerDay = sum(steps))
    ggplot(data = dailySteps2, aes(x = stepsPerDay)) + geom_histogram() + labs(x = "Number of steps per day") 

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](Pm1_template_files/figure-markdown_strict/recalculate%20daily%20number%20of%20steps-1.png)

    medianSteps <- median(dailySteps2$stepsPerDay)
    medianSteps

    ## [1] 10766.19

    meanSteps <- mean(dailySteps2$stepsPerDay)
    meanSteps

    ## [1] 10766.19

The median number of steps per day is 10766. The mean number of steps is
10766.

Overall, the changes between median and mean calculated by omitting
missing values versus including imputed values are very minor:

Median (without NA): 10765  
Median (with imputed values): 10766

Mean (without NA): 10766  
Mean (with imputed values): 10766

### Are there differences in activity patterns between weekdays and weekends?

I will use the weekdays() function to create a new factor variable
"dayType" in newDataset. DayType has two levels - "weekday" and
"weekend" indicating whether a given date is a weekday or weekend day.

    weekend <- weekdays(as.Date(newDataset$date)) %in% c("Saturday", "Sunday")
    newDataset$dayType <- "weekday"
    newDataset$dayType[weekend == TRUE] <- "weekend"
    newDataset$dayType <- as.factor(newDataset$dayType)

    ## Day 1 of the experiment was Monday, October 1 2012 (google confirms that it was a Monday). Each day has 288 intervals (12x24x5min).

Finally, I will make a panel plot containing a time series plot of the
5-minute interval (x-axis) and the average number of steps taken,
averaged across all weekday days or weekend days (y-axis).

    newIntervalSteps <- aggregate(steps ~ interval + dayType, newDataset, mean)
    library(lattice)
    xyplot(
            steps ~ interval | dayType,
            newIntervalSteps,
            type = "l",
            layout = c(1,2),
            main = "Time Series Plot",
            xlab = "Interval",
            ylab = "Average Number of Steps"
    )

![](Pm1_template_files/figure-markdown_strict/calculate%20average%20number%20of%20per%20intervall%20and%20weekday%20or%20weekend-1.png)
