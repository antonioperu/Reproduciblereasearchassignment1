### Loading data

    setwd("C:/Users/anton/Desktop")
    activity <- read.csv("activity.csv")

**What is mean total number of steps taken per day?**
=====================================================

Total number of steps each day
------------------------------

    stepsDay <- aggregate(steps ~ date, data = activity, FUN = sum, na.rm = TRUE)

Histogram
---------

    hist(stepsDay$steps, xlab = "# of Steps/Day", col = "red")

![](ReproducibleResearch1_files/figure-markdown_strict/unnamed-chunk-3-1.png)

Mean and Median
---------------

    mean_steps <- mean(stepsDay$steps)
    median_steps <- median(stepsDay$steps)
    cat("mean steps taken each day is ", mean_steps)

    ## mean steps taken each day is  10766.19

    cat("median steps taken each day is ", median_steps)

    ## median steps taken each day is  10765

\#\*\* What is the average daily activity pattern?\*\*

\#\#Make a time series plot (i.e. type = “l”) of the 5-minute interval
(x-axis) and the average number of steps taken, averaged across all days
(y-axis)

    meanActivitySteps <- aggregate(steps ~ interval, data = activity, FUN = mean, na.rm = TRUE)

    plot(meanActivitySteps$interval, meanActivitySteps$steps, type = "l", col = "tan3", xlab = "Intervals", ylab = "Steps/Interval")

![](ReproducibleResearch1_files/figure-markdown_strict/unnamed-chunk-5-1.png)

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
-------------------------------------------------------------------------------------------------------------

    maxSteps <-max(meanActivitySteps$steps)
    maxInterval <- meanActivitySteps$interval[which(meanActivitySteps$steps == maxSteps)]

    maxSteps

    ## [1] 206.1698

    maxInterval

    ## [1] 835

\#**Inputing missing values**

\#\#Calculate and report the total number of missing values in the
dataset (i.e. the total number of rows with NAs)

    sum(is.na(activity))

    ## [1] 2304

\#\#Devise a strategy for filling in all of the missing values in the
dataset

    missingValues <- subset(activity, is.na(steps))

    par(mfrow = c(2,1), mar = c(2, 2, 1, 1))
    hist(missingValues$interval, main="NA/Interval")
    hist(as.numeric(missingValues$date, main="NA/Date"))

![](ReproducibleResearch1_files/figure-markdown_strict/unnamed-chunk-8-1.png)

\#\#Create a new dataset that is equal to the original dataset but with
the missing data filled in.

    MeanStepsPerInterval <- tapply(activity$steps, activity$interval, mean, na.rm = TRUE)
    activityNAs <- activity[is.na(activity$steps),]
    activityNonNAs <- activity[!is.na(activity$steps),]
    activityNAs$steps <- as.factor(activityNAs$interval)
    levels(activityNAs$steps) <- MeanStepsPerInterval
    levels(activityNAs$steps) <- round(as.numeric(levels(activityNAs$steps)))
    activityNAs$steps <- as.integer(as.vector(activityNAs$steps))
    imputedActivity <- rbind(activityNAs, activityNonNAs)

\#\#Make a histogram of the total number of steps taken each day and
Calculate and report the mean and median total number of steps taken per
day. Do these values differ from the estimates from the first part of
the assignment? What is the impact of imputing missing data on the
estimates of the total daily number of steps?

    par(mfrow = c(1,2))

    activityStepsDay <- aggregate(steps ~ date, data = activity, FUN = sum, na.rm = TRUE)
    hist(activityStepsDay$steps, xlab = "Steps/Day", main = "steps/day", col = "red")

    impActivityStepsDay <- aggregate(steps ~ date, data = imputedActivity, FUN = sum, na.rm = TRUE)
    hist(impActivityStepsDay$steps, xlab = "Steps/Day", main = "NAs IMPUTED - steps/day", col = "red")

![](ReproducibleResearch1_files/figure-markdown_strict/unnamed-chunk-10-1.png)

\#**Are there differences in activity patterns between weekdays and
weekends?**

\#\#Create a new factor variable in the dataset with two levels –
“weekday” and “weekend” indicating whether a given date is a weekday or
weekend day.\#\#

    imputedActivity$dayType <- ifelse(weekdays(as.Date(imputedActivity$date)) == "Saturday" | weekdays(as.Date(imputedActivity$date)) == "Sunday", "weekend", "weekday")
    #transform dayType variable into factor
    imputedActivity$dayType <- factor(imputedActivity$dayType)

\#\#Make a panel plot containing a time series plot (i.e. type = “l”) of
the 5-minute interval (x-axis) and the average number of steps taken,
averaged across all weekday days or weekend days (y-axis). See the
README file in the GitHub repository to see an example of what this plot
should look like using simulated data.\#\#

    stepsIntervalDayType <- aggregate(steps ~ interval + dayType, data = imputedActivity, FUN = mean)

    head(stepsIntervalDayType)

<script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["interval"],"name":[1],"type":["int"],"align":["right"]},{"label":["dayType"],"name":[2],"type":["fctr"],"align":["left"]},{"label":["steps"],"name":[3],"type":["dbl"],"align":["right"]}],"data":[{"1":"0","2":"weekday","3":"1.75409836","_rn_":"1"},{"1":"5","2":"weekday","3":"0.29508197","_rn_":"2"},{"1":"10","2":"weekday","3":"0.11475410","_rn_":"3"},{"1":"15","2":"weekday","3":"0.13114754","_rn_":"4"},{"1":"20","2":"weekday","3":"0.06557377","_rn_":"5"},{"1":"25","2":"weekday","3":"2.08196721","_rn_":"6"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>

    names(stepsIntervalDayType) <- c("interval", "dayType", "meanSteps")

    library(ggplot2)

    ## Warning: package 'ggplot2' was built under R version 3.6.3

    plot <- ggplot(stepsIntervalDayType, aes(interval, meanSteps))
    plot + geom_line(color = "green") + facet_grid(dayType~.) + labs(x = "Intervals", y = "Average Steps", title = "Activity Patterns")

![](ReproducibleResearch1_files/figure-markdown_strict/unnamed-chunk-12-1.png)
