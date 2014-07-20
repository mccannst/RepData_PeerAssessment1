# Reproducible Research: Peer Assessment 1
========================================================

## Loading and preprocessing the data
1. Loading and Processing Data
Using the cloned directory from GitHub, unzip and read
the activity data into a data frame in the environment


```r
library(markdown)
library(plyr)
library(knitr)
library(ggplot2)

unzip("activity.zip")
activity0 <- read.csv("activity.csv")
```


## What is mean total number of steps taken per day?
Using plyr, we add the total number of steps per day
across all intervals, and show those values

```r
stepstotal <- ddply(activity0, .(date), summarize, steps = sum(steps))
```

First, a Histogram of the total steps:

```r
hist(stepstotal$steps, breaks = 20)
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 


Then, our computed mean and median:

```r
mean(stepstotal$steps, na.rm = TRUE)
```

```
## [1] 10766
```

```r
median(stepstotal$steps, na.rm = TRUE)
```

```
## [1] 10765
```


## What is the average daily activity pattern?
We aggregate the data by interval and use the mean function
to gather the mean values for each five-minute interval
across all dates.

```r
avgsteps <- aggregate(. ~ interval, FUN = mean, data = activity0)
```



```r
plot(avgsteps$interval, avgsteps$steps, type = "l")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 


## Which 5-minute interval, on average across all the days
## in the dataset, contains the maximum number of steps?


```r
max(avgsteps$steps)
```

```
## [1] 206.2
```

```r
topinterval <- subset(avgsteps, steps > 206)
topinterval$interval
```

```
## [1] 835
```


## Imputing missing values

1. We build a function to replace the NAs in a data set with
the means of the other values for that observation.

Built on the practice listed here at [Stack Overflow] (http://stackoverflow.com/questions/9322773/how-to-replace-na-with-mean-by-subset-in-r-impute-with-plyr/9322975#9322975)


```r
impute.mean <- function(x) replace(x, is.na(x), mean(x, na.rm = TRUE))
```

2. Then we apply that function to our data set to make a new data set with imputed values:


```r
activity.imputed <- plyr::ddply(activity0[1:3], .(interval), transform, steps = impute.mean(steps), 
    date = date, interval = interval)
activity.imputed <- activity.imputed[order(activity.imputed$date), ]
stepstotal.imputed <- ddply(activity.imputed, .(date), summarize, steps = sum(steps))
```


Histogram of average steps with imputed data:

```r
hist(stepstotal.imputed$steps, breaks = 20)
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10.png) 


What impact did imputation have on the means and medians of the data set?


```r
mean.imputed.steps <- mean(stepstotal.imputed$steps)
median.imputed.steps <- median(stepstotal.imputed$steps)
mean.steps <- mean(stepstotal$steps, na.rm = TRUE)
median.steps <- median(stepstotal$steps, na.rm = TRUE)
```

Mean of imputed steps = 

```r
mean.imputed.steps
```

```
## [1] 10766
```

Median of imputed steps = 

```r
median.imputed.steps
```

```
## [1] 10766
```


Mean of original data set steps = 

```r
mean.steps
```

```
## [1] 10766
```

And median of original data set steps = 

```r
median.steps
```

```
## [1] 10765
```

We find that using this method (replacing NAs with the
means of the values for intervals before) doesn't change
the mean or median one bit. Other methods will produce other values for the mean and median in imputed data sets, however.

## Are there differences in activity patterns between weekdays and weekends?

1. create a new column in the dataset with two levels for weekday and weekend. 


```r
activity.imputed$weekpart <- "weekday"
```


2. then, set up the logical test vector for weekends


```r
weekdaysample <- weekdays(as.Date(activity.imputed$date))
weekendsample <- weekdaysample %in% c("Saturday", "Sunday")
```


3. Finally, change the weekpart column to match the weekendsample test vector


```r
activity.imputed$weekpart[weekendsample == TRUE] <- "weekend"
```


Then plot. The plot below shows the differences between Weekday activity intervals and weekend activity intervals, specifically, the weekdays show much more activity between intervals 500 and 1000 than the weekend activity logs.


```r
qplot(interval, steps, data = activity.imputed, facets = . ~ weekpart, geom = "line")
```

![plot of chunk unnamed-chunk-19](figure/unnamed-chunk-19.png) 



