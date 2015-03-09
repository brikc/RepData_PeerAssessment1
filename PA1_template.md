# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

The data file was included in the repository, so this script assumes the zip file is in the working directory.  Unzip if needed, and read the CSV into the system.
    

```r
if (!file.exists("activity.csv")) {
    unzip("activity.zip")
} 
d <- read.csv("activity.csv")
```
This stores in variable "d" a data.frame of 3 variables and 17568 observations.  For the first several exercises, we want to work with a dataset that has removed all rows with NA values.  So we create a new dataset, called "fulldata", with all NA rows removed like so:


```r
fulldata<-d[!is.na(d$steps),]
```

## What is mean total number of steps taken per day?

First we calculate the total number of steps taken per day.  This is done by taking an aggregate by date and applying the "sum" function on the steps variable. We specify "TRUE" for na.rm to ignore NA values.  Finally, in this step we assign more meaningful column names to the totalByDay object: 


```r
totalByDay <- aggregate(fulldata$steps, by=list(fulldata$date), FUN = sum)
names(totalByDay) <- c("date","totalsteps")
```

Next, we create a histogram of the total number of steps taken each day.  (A note on histograms and barplots: The difference between a barplot and a histogram is that a barplot is used for displaying distributions of categorical variables while histograms are used for
numerical variables.  The axis in a histogram is a number line representing the numeric variables, and hence the orders of
the bars cannot be changed on a histogram.  In a bar plot the categories can be listed in any order, though there is often an ordering that makes more sense than others.)

Here is a histogram showing the distribution of total steps per day.


```r
hist(totalByDay$totalsteps,col="cornflowerblue",xlab="Total Steps per Day",breaks=20,main="Histogram of Total Steps per Day")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 


Next we calculate and report the mean and median of the total number of steps taken per day.  This is done using the mean() and median() functions on our object "totalByDay".


```r
mean(totalByDay$totalsteps)
```

```
## [1] 10766.19
```

```r
median(totalByDay$totalsteps)
```

```
## [1] 10765
```


## What is the average daily activity pattern?

Next we will make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis).  This is done in several steps.

First, we create a new data.frame by aggregating on interval and applying the mean function to the number of steps for each interval.  The second line of code assigns more meaningful column names to the new data.frame.


```r
avgByInterval <- aggregate(fulldata$steps, by=list(fulldata$interval), FUN = mean, na.rm = TRUE)
names(avgByInterval) <- c("interval","meansteps")
```

If we create a plot based on this new table, gaps will appear in the plot based on the existing intervals.  For example, the first hour is labeled in 5 minute increments, but when it gets to the second hour, it jumps from 55 to 100.  See rows 12 and 13 below:


```r
head(avgByInterval,14)
```

```
##    interval meansteps
## 1         0 1.7169811
## 2         5 0.3396226
## 3        10 0.1320755
## 4        15 0.1509434
## 5        20 0.0754717
## 6        25 2.0943396
## 7        30 0.5283019
## 8        35 0.8679245
## 9        40 0.0000000
## 10       45 1.4716981
## 11       50 0.3018868
## 12       55 0.1320755
## 13      100 0.3207547
## 14      105 0.6792453
```

So, to fix this we convert the given intervals to actual units of time.  This will give us a plot that's distributed evenly throughout the 24 hour period. This conversion is accomplished by dividing each interval by 100, printing the result to 2 decimal places, and then using strptime to read in the printed value based on its format:



```r
avgByInterval$time <- strptime(sprintf("%02.2f", avgByInterval$interval / 100), format="%H.%M")
```

Here is the resulting plot: 


```r
plot(avgByInterval$time,avgByInterval$meansteps,type="l",main="Average Steps by Interval for All Days",xlab="Interval",ylab="Average Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png) 


Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps? To answer this, we subset our data.frame based on the max value for average interval:


```r
subset(avgByInterval,subset=avgByInterval$meansteps==max(avgByInterval$meansteps))
```

```
##     interval meansteps                time
## 104      835  206.1698 2015-03-09 08:35:00
```

So the maximum average steps occured at interval 835 (or, in other words, at 8:35am).

## Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

So next, we calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs).  To do this, we take the sum of the result of applying "is.na" to the data.frame. From that, we now know there are 2304 observations with missing values:


```r
sum(is.na(d))
```

```
## [1] 2304
```

Next, we must devise a strategy for filling in all of the missing values in the dataset. According to the assignment, the strategy does not need to be sophisticated. For example, we can use the mean/median for that day, or the mean for that 5-minute interval, etc.

I decided to use the mean for that interval.  One reason I decided to go with this route is because some days are actually completed missing values, and so not everyday would have a  mean to use.

I create a new dataset that is equal to the original dataset but with the missing data filled in with the mean for that day.

Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

## Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.
