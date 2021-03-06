# Reproducible Research: Peer Assessment 1

## Loading and preprocessing the data

Make sure the `data.table` package is loaded and then load the raw data from `activity.csv`  


```r
require(data.table)||install.packages(data.table)
actData<-data.table(read.csv("activity.csv"))
```

## What is mean total number of steps taken per day?


*1.  Calculate the total number of steps taken per day*  

This is grouped together in dailyData, which is actData grouped by date

```r
dailyData<-actData[, list(steps=sum(steps, na.rm=TRUE)), by=date]
```

*2.  If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day*

The histogram of total steps per day is below:


```r
hist(dailyData$steps, main="Histogram of Daily Steps", xlab="Total Steps per Day")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 


*3.  Calculate and report the mean and median of the total number of steps taken per day*


```r
format(mean(dailyData$steps), scientific=FALSE)
```

Mean steps per day are: 9354.23


```r
format(median(dailyData$steps), scientific=FALSE)
```

Median steps per day are: 10395


## What is the average daily activity pattern?

*1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)*

Time series plot is below:  

```r
intervalData<-actData[, list(steps=mean(steps, na.rm=TRUE)), by=interval]
plot(intervalData$steps ~ intervalData$interval, type="l", xlab="Interval", ylab="Steps", xaxt="n")
axis(side=1, at=seq(from=0, to=2400, by=300))
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png) 

*2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?*
The 5 minute interval with the max number of steps on average across all days can be found by:

```r
intervalData[intervalData[,intervalData$steps==max(intervalData$steps)]]
```

```
##    interval    steps
## 1:      835 206.1698
```

## Imputing missing values
*Note that there are a number of days/intervals where there are missing values (coded as `NA`). The presence of missing days may introduce bias into some calculations or summaries of the data.*

*1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with `NAs`)*


```r
completes<-complete.cases(actData)
sum(!completes)
```

```
## [1] 2304
```


*2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.*

Will take the rounded mean for that 5-minute interval

*3. Create a new dataset that is equal to the original dataset but with the missing data filled in.*

This takes some explanation.  

a. Merge `intervalData` with `actData` to create a frame which displays both the missing data and the mean values.  There will be two step fields, `steps.x` from `actData` and `steps.y` from `intervalData`.  
b. Where `steps.x` is `NA`, copy across from `steps.y`
c. Remove `steps.y` and rename `steps.x` to `steps`.


```r
filledData<-merge(actData,intervalData,by="interval")
filledData[is.na(steps.x), steps.x:=round(steps.y)]
filledData<-filledData[,list(steps=steps.x, date=date, interval=interval)]
```

*4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?*


```r
filledDailyData<-filledData[, list(steps=sum(steps)), by=date]
hist(filledDailyData$steps, main="Histogram of Daily Steps (Imputed)", xlab="Total Steps per Day (Imputed)")
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png) 


```r
format(mean(filledDailyData$steps), scientific=FALSE)
```

Mean steps per day (imputed) are: 10765.64


```r
format(median(filledDailyData$steps), scientific=FALSE)
```

Median steps per day (imputed) are: 10762

The mean and median steps per day for the imputed data set differ from the mean and median steps per day for base data set.  

The impact of imputing missing data is to increase the mean and median total daily number of steps compared to the base data set.  The frequency distribution has also changed;  
a.  There are fewer days with less than 5000 steps and more days with between 10000 and 15000 steps.  
b.  The mean and median have converged.  

## Are there differences in activity patterns between weekdays and weekends?

*For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.*

*1  Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.*


```r
weekendDays<-c("Saturday", "Sunday")
filledData$weekend<-factor(weekdays(as.Date(filledData$date)) %in% weekendDays, levels=c(TRUE, FALSE), labels=c("Weekend", "Weekday"))
```

*2  Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.*


```r
weekends<-filledData[,filledData$weekend=="Weekend"]
weekendData<-filledData[weekends]
weekdayData<-filledData[!weekends]

weekendData<-weekendData[,list(steps=mean(steps)), by=interval]
weekdayData<-weekdayData[,list(steps=mean(steps)), by=interval]

maxSteps<-max(weekendData$steps)
if (maxSteps < max(weekdayData$steps)) {maxSteps<-max(weekdayData$steps)}

#Plot the weekend data
layout(matrix(1:2,ncol=1), widths=1)

plot(weekendData$steps~weekendData$interval, type="l", main="Weekend", xlab="Interval", ylab="Steps", xaxt="n", yaxt="n", ylim=c(0,maxSteps))
axis(side=1, at=seq(from=0, to=2400, by=300))
axis(side=2, at=seq(from=0, to=maxSteps, by=100))

plot(weekdayData$steps~weekdayData$interval, type="l", main="Weekday", xlab="Interval", ylab="Steps", xaxt="n", yaxt="n", ylim=c(0,maxSteps))
axis(side=1, at=seq(from=0, to=2400, by=300))
axis(side=2, at=seq(from=0, to=maxSteps, by=100))
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png) 

Yes, there are differences in activity patterns.
