# Reproducible Research: Peer Assessment 1 


## Loading and preprocessing the data
Ddata will be directly loded in from the files provided in the repo using the following code.


```r
unzip("activity.zip")

raw_data_set<-read.csv("activity.csv", na.strings = "NA")
```

Once the dataset has been loaded the next step is to convert and load the data into the following datasets 

```r
raw_data_set$interval <- sprintf("%04d", raw_data_set$interval)

raw_data_set$date <- as.Date(raw_data_set$date)
```


## What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day Make a histogram of the total number of steps taken each day:

```r
sum_by_day <- aggregate(raw_data_set$steps, by = list(raw_data_set$date),
                        sum, na.rm=TRUE)

hist(sum_by_day$x, breaks = 10, xlab="Total number of steps per day", main="Histogram of total steps per day", col="lightskyblue")
```

![](PA1_template_files/figure-html/unnamed-chunk-16-1.png)<!-- -->

2. Calculate and report the mean and median of the total number of steps taken per day

```r
total_mean <- mean(sum_by_day$x)

total_median <- median(sum_by_day$x)
```

|Measurement|Value|
|-----------|-----|
|Mean|9354|
|Median|10395|


## What is the average daily activity pattern?

1. Make a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
mean_by_interval <- aggregate(raw_data_set$steps, by = list(raw_data_set$interval),                     mean, na.rm=TRUE)

plot(mean_by_interval, type='l', main="Mean Daily Pattern", xlab="Interval",         ylab=" mean steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-18-1.png)<!-- -->

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
The interval with the maximum mean steps is:


```r
subset(mean_by_interval,x==max(x))[1,1]
```

```
## [1] "0835"
```
## Imputing missing values

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with 𝙽𝙰s)

```r
sum(is.na(raw_data_set))
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
filled_dataset <- raw_data_set

filled_dataset$steps <- apply(filled_dataset,1,function(x){
    if(is.na(x[1])){ 
          x=as.integer(mean_by_interval[mean_by_interval$Group.1==x[3],2]) 
    } else as.integer(x[1])})

sum(is.na(filled_dataset))
```

```
## [1] 0
```

3. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. 


```r
sum_by_day_filled <- aggregate(filled_dataset$steps, by = list(filled_dataset$date), sum, na.rm=TRUE)

hist(sum_by_day_filled$x, breaks = 10, xlab="Total number of steps per day", main="Histogram of total steps per day with filled NAs", col="lightskyblue")
```

![](PA1_template_files/figure-html/unnamed-chunk-22-1.png)<!-- -->

4. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
total_mean<-mean(sum_by_day_filled$x)

total_median<-median(sum_by_day_filled$x)
```
|Measurement|Value|
|-----------|-----|
|Mean|10749|
|Median|10641|

Now that to the estimation of NAs values are taken into account the mean and median values appear  more similar than the previous dataset without NA values. 

## Are there differences in activity patterns between weekdays and weekends?

Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
filled_dataset['day_type'] <- apply(filled_dataset,1,function(x){if (weekdays(as.Date(x[2])) %in% c('sábado','domingo')){x[4]='weekEnd'} else {x[4]='weekDay'}})
```
Make a panel plot containing a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
library(lattice)

mean_by_interval_and_days <- aggregate(filled_dataset$steps, by = list(filled_dataset$interval, filled_dataset$day_type), mean)

mean_by_interval_and_days$Group.1 <- as.integer(mean_by_interval_and_days$Group.1)
xyplot(x~Group.1|Group.2, data = mean_by_interval_and_days,
    type = 'l',
    xlab = 'Interval',
    ylab = 'Number of Steps',
    layout = c(1,2))
```

![](PA1_template_files/figure-html/unnamed-chunk-25-1.png)<!-- -->

According to the results the studies user normally get up early in the week days ( around 5am) and have a peak of activity around 8.30am. This peak could be produced by a morning walk to the work. In week days between 9am and 6pm aprroximatelly the pattern is quite stable probabbly is a pattern derived from a non physical work. After 6pm in week days the user has and activty increase, probably due to return to home.

This is not true.  on weekend the users typically starts the day around 6am to 7 am. Simmilairly to weekdays at around 8.30 a peak period begins but it is similar to the peak during weekdays. That being said throughout the day the pattern is more random than during weekdays. This could be a result of less constraint imposed by obligations like work. Lastly the time they go to sleep on weekends is later than the week days. 
