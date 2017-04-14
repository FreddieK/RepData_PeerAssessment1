# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data




```r
data <- read.csv(unzip('activity.zip'))
```

## What is mean total number of steps taken per day?

1. Make a histogram of the total number of steps taken each day

```r
byDay = with(data, aggregate(steps ~ date, FUN = sum))
hist(byDay$steps)
```

![](PA1_template_files/figure-html/hist-1.png)<!-- -->

2. Calculate and report the mean and median total number of steps taken per day


```r
byDay$steps %>% mean(na.rm=TRUE)
```

```
## [1] 10766.19
```

```r
byDay$steps %>% median(na.rm=TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
byInterval = with(data, aggregate(steps ~ interval, FUN = mean))

with(byInterval, plot(interval, steps, type = 'l'))
```

![](PA1_template_files/figure-html/timeSeries-1.png)<!-- -->

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
byInterval %>% filter(steps == max(steps)) %>% select(interval)
```

```
##   interval
## 1      835
```

## Imputing missing values

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
data %>% filter(is.na(steps)) %>% count()
```

```
## # A tibble: 1 x 1
##       n
##   <int>
## 1  2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
# Let's impute by Interval mean
imputed = data
imputed$steps = with(imputed, ifelse(!is.na(steps), steps, byInterval[byInterval$interval == interval,]$steps))
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
imputedByDay = with(imputed, aggregate(steps ~ date, FUN = sum))
hist(imputedByDay$steps)
```

![](PA1_template_files/figure-html/histogramWithImputedValues-1.png)<!-- -->

```r
imputedByDay$steps %>% mean()
```

```
## [1] 10766.19
```

```r
imputedByDay$steps %>% median()
```

```
## [1] 10765.59
```

No impactful change on mean or median when imputing based on interval. The difference is that we get an estimate for steps for 2012-10-01, which previously was lacking.

## Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
imputed = mutate(imputed, 
       weekpart = ifelse(weekdays(as.Date(date)) %in% c('Saturday', 'Sunday'), 
                         'weekend', 
                         'weekday'
                         )
       )
```

1. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The plot should look something like the following, which was created using simulated data:


```r
byIntervalAndWeekpart = with(imputed, aggregate(steps ~ interval + weekpart, FUN = mean))

par(mfrow=c(2,1))

with(byIntervalAndWeekpart[byIntervalAndWeekpart$weekpart == "weekend",], 
     plot(interval, steps, type = 'l', main = "Weekend"))
with(byIntervalAndWeekpart[byIntervalAndWeekpart$weekpart == "weekday",], 
     plot(interval, steps, type = 'l', main = "Weekday"))
```

![](PA1_template_files/figure-html/plotIntervalByWeekpart-1.png)<!-- -->
