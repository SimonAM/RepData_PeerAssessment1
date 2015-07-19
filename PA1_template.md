# Reproducible Research: Peer Assessment 1


Dplyr is used during the loading and preprocessing the data



```r
setwd("C:/Users/Simon/Google Drive/R/coursera/reproducible research/RepData_PeerAssessment1")
unzip("activity.zip")
activity <- read.csv("activity.csv")
suppressPackageStartupMessages(library(dplyr))
```


## What is mean total number of steps taken per day?

Total steps per day

```r
activity %>% 
        group_by(date) %>% 
        summarize(total=sum(steps)) %>% 
        print
```

```
## Source: local data frame [61 x 2]
## 
##          date total
## 1  2012-10-01    NA
## 2  2012-10-02   126
## 3  2012-10-03 11352
## 4  2012-10-04 12116
## 5  2012-10-05 13294
## 6  2012-10-06 15420
## 7  2012-10-07 11015
## 8  2012-10-08    NA
## 9  2012-10-09 12811
## 10 2012-10-10  9900
## ..        ...   ...
```



```r
hist(activity$steps, xlab="Steps", main="Histogram of overall steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

Mean and median number of steps per day

```r
activity %>% 
        group_by(date) %>% 
        summarize(mean=mean(steps, na.rm=TRUE), median=median(steps, na.rm=TRUE)) %>% 
        print
```

```
## Source: local data frame [61 x 3]
## 
##          date     mean median
## 1  2012-10-01      NaN     NA
## 2  2012-10-02  0.43750      0
## 3  2012-10-03 39.41667      0
## 4  2012-10-04 42.06944      0
## 5  2012-10-05 46.15972      0
## 6  2012-10-06 53.54167      0
## 7  2012-10-07 38.24653      0
## 8  2012-10-08      NaN     NA
## 9  2012-10-09 44.48264      0
## 10 2012-10-10 34.37500      0
## ..        ...      ...    ...
```


## What is the average daily activity pattern?

```r
interval_orig <-activity %>%
        group_by(interval) %>%
        summarize(meanint=mean(steps, na.rm=T))
order_int <- interval_orig %>%
        arrange(desc(meanint))
        
max_steps <- order_int[[1]][[1]]
```
The 5 minute interval with the highest number of steps is 835  as seen in the time series plot below.

```r
plot(interval_orig$interval,interval_orig$meanint, type="l", xlab="interval", ylab="Mean steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png) 




## Imputing missing values

```r
missnum <- activity %>%
        is.na %>%
        sum
```
Nr of missing values are 2304



```r
missingsteps <- is.na(activity$steps)
activity$steps[missingsteps] <- mean(activity$steps,na.rm=T)

impute_activity <- data.frame(activity)
activity <- read.csv("activity.csv")
```


```r
total_impute <- impute_activity %>% 
        group_by(date) %>% 
        summarize(total=sum(steps))
hist(total_impute$total,xlab="Total steps", main="Histogram of total steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png) 

Mean & median steps per day

```r
impute_activity %>% 
        group_by(date) %>% 
        summarize(mean=mean(steps, na.rm=TRUE), median=median(steps, na.rm=TRUE)) %>% 
        print
```

```
## Source: local data frame [61 x 3]
## 
##          date     mean  median
## 1  2012-10-01 37.38260 37.3826
## 2  2012-10-02  0.43750  0.0000
## 3  2012-10-03 39.41667  0.0000
## 4  2012-10-04 42.06944  0.0000
## 5  2012-10-05 46.15972  0.0000
## 6  2012-10-06 53.54167  0.0000
## 7  2012-10-07 38.24653  0.0000
## 8  2012-10-08 37.38260 37.3826
## 9  2012-10-09 44.48264  0.0000
## 10 2012-10-10 34.37500  0.0000
## ..        ...      ...     ...
```

Does these numbers differ from the first part of the assignment? Yes.The imputation increased the proportion of low activity compared to the non-imputed values seen below.

```r
activity %>% 
        group_by(date) %>% 
        summarize(mean=mean(steps, na.rm=TRUE), median=median(steps, na.rm=TRUE)) %>% 
        print
```

```
## Source: local data frame [61 x 3]
## 
##          date     mean median
## 1  2012-10-01      NaN     NA
## 2  2012-10-02  0.43750      0
## 3  2012-10-03 39.41667      0
## 4  2012-10-04 42.06944      0
## 5  2012-10-05 46.15972      0
## 6  2012-10-06 53.54167      0
## 7  2012-10-07 38.24653      0
## 8  2012-10-08      NaN     NA
## 9  2012-10-09 44.48264      0
## 10 2012-10-10 34.37500      0
## ..        ...      ...    ...
```

## Are there differences in activity patterns between weekdays and weekends?


```r
stripped_date <- strptime(impute_activity$date,format="%Y-%m-%d")
daysofweek <- weekdays(stripped_date)
weekends <- daysofweek=="Saturday"|daysofweek=="Sunday"
workdays <- daysofweek=="Monday"|daysofweek=="Tuesday"|daysofweek=="Wednesday"|daysofweek=="Thursday"|daysofweek=="Friday"
weekly_activity <- data.frame(impute_activity, workdays, weekends)
```


```r
weekend_steps <- weekly_activity[weekends==TRUE,] %>% group_by(interval) %>% summarize(mean=mean(steps,na.rm=T))
workday_steps <- weekly_activity[weekends==FALSE,] %>% group_by(interval) %>% summarize(mean=mean(steps,na.rm=T))
par(mfcol=c(2,1))
plot(workday_steps$interval,workday_steps$mean, type="l", main="Workdays", xlab="Interval", ylab="Number of steps")
plot(weekend_steps$interval,weekend_steps$mean, type="l", main="Weekends", xlab="Interval", ylab="Number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png) 
