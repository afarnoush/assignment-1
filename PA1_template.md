Reproducible Research: Peer Assessment 1
========================================

### Loading and preprocessing the data

#### 1. Load the data (i.e. read.csv())

Read in the data from the local zipped file to the object activty, and
convert the activity object to a tbl class.

    activity <- read.csv('activity.csv')## read in the data
    activity <- tbl_df(activity) ## structure the data as a tbl class

#### 2.What is mean total number of steps taken per day?

First, the number of steps per day aggregated. The group\_by function
and then the summarise function from dplyr is used to perform the
aggregation of steps by day and last the hist function to create the
histogram plot.

    activity_days <- activity %>% group_by(date) %>% summarise(total.steps = sum(steps))
    hist(activity_days$total.steps, breaks = 25,col = "darkblue", main = "Histogram of Total Steps per Day",xlab = "Number of Steps per Day", ylab="Number of times (Count)")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-3-1.png)

#### 3.Report the mean total number of steps taken per day

    mean((activity_days$total.steps),na.rm= TRUE)

    ## [1] 10766.19

    median((activity_days$total.steps),na.rm= TRUE)

    ## [1] 10765

#### 4.What is the average daily activity pattern?

Create a factor of the interval - time of day - so that we can aggregate

    activity$interval.f <- as.factor(activity$interval)

Calculate the average number of steps for each interval using the
group\_by and summarise functions

    activity_interval<- activity %>% group_by(interval.f) %>% summarise(mean.steps = mean(steps, na.rm =TRUE))

#### 5.Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

    activity_interval$interval <- as.numeric(as.character(activity_interval$interval.f))

    plot(activity_interval$interval, activity_interval$mean.steps, type = "b", xaxt="n", xlab = "5-minute interval"     , ylab = "mean steps", main = "Daily Activity Pattern")
       axis(1, at = seq(100, 2300, by = 100), las = 2)

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-7-1.png)

#### 6. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

    max_steps_interval <- which.max(activity_interval$mean.steps)
    print(activity_interval[max_steps_interval,])

Source: local data frame [1 x 3]

interval.f mean.steps interval (fctr) (dbl) (dbl) 1 835 206.1698 835

### Imputing missing values:

#### 1.Total number of missing values in the dataset:

    sum(is.na(activity$steps))

    ## [1] 2304

I used impute function from the Hmisc library to create a new dataset
that is equal to the original dataset but with the missing data filled
in.

#### 2.Fill in the missing data

    activityDataImputed <- activity
    activityDataImputed$impute.steps <- impute(activity$steps, mean)

#### 3.Make a histogram of the total number of steps taken each day

    stepsByDayImputed <- tapply(activityDataImputed$impute.steps, activityDataImputed$date, sum)
    hist(stepsByDayImputed,breaks = 25,col = "darkred",ylim = range(0:20), xlab="Number of Steps per Day (Imputed)", ylab="Number of times (Count)",plot= TRUE)

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-11-1.png)

#### 4.Calculate and report the mean and median total number of steps taken per day

    mean(stepsByDayImputed)

    ## [1] 10766.19

    median(stepsByDayImputed)

    ## [1] 10766.19

#### 5.Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

The impact of inputting missing data is minimal, as the main remain
constant and only median seems to be changing by just over one step.The
shape of the both histograms remains the same. However, the frequency
counts increased as expected.

### Are there differences in activity patterns between weekdays and weekends?

    activityDataImputed$dateType <-  ifelse(as.POSIXlt(activityDataImputed$date)$wday %in% c(0,6), 'weekend', 'weekday')

#### Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

    activityDataImputed_mean <- aggregate(steps ~ interval + dateType, data=activityDataImputed, mean)

    xyplot(steps ~ interval| dateType, data = activityDataImputed_mean, 
          type = "l", layout = c(1,2), xlab = "5-minute Interval",xlim=range(0:2500), ylab = "Number of Steps", 
          main = "Average Steps by 5-minute Interval for Weekends and Weekdays")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-14-1.png)

There are clear differences in activity between weekends and weekdays,
as most people are more active in the weekends than they are during the
week.However, the graph shows that activity on the weekday has the
greatest peak from all steps intervals.
