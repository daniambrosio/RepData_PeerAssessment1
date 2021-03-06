# Reproducible Research: Peer Assessment 1
Daniel Rodrigues Ambrosio  

***Note:*** URL for the HTML version of this document: http://rpubs.com/daniambrosio/coursera-datascience-repdata-project1


This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.


## Loading and preprocessing the data
The chunck of R Code below should download a zip file from its original location into a data folder.
After downloading it, unzip it into the same data folder. 

Original location of the zip file: https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip


```r
# variables to handle path, dir and file names
dir.data <- paste(getwd(),"data",sep="/")
file.csv <- paste(dir.data,"activity.csv",sep="/")
zipFile <- paste(dir.data,"projectdata.zip",sep="/")
# if data folder does not exist, create it
if(!file.exists(dir.data)) {dir.create(dir.data)}
# donwload zip file only if it does not exist locally
if(!file.exists(zipFile)) {
    zipFileURL <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
    download.file(zipFileURL, destfile=zipFile, method="curl")
}
# unzip file only if the raws csv file cannot be found
if(!file.exists(file.csv)) {
    unzip(zipFile, exdir=dir.data)
} 
# read the CSV file
activity <- read.csv(file.csv)
str(activity)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```


## What is mean total number of steps taken per day?

1. Make a histogram of the total number of steps taken each day

The following code will aggregate all the steps per day and store it in a new variable.
Then this new variable is used to plot a bar graph having the date in the x axis and the amount of steps in the y axis.



```r
steps.date <- aggregate(steps~date,activity,sum)
hist(steps.date$steps, xlab="Total steps by day", ylab="Frequency [Days]",main="Histogram : Number of daily steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png) 

2. Calculate and report the **mean** and **median** total number of steps taken per day

The values will be calculated and stored in variables to be compared further in the exercise.


```r
mean1 <- mean(steps.date$steps, na.rm=TRUE)
median1 <- median(steps.date$steps, na.rm=TRUE)
```


```
## [1] 10766.19
```

```
## [1] 10765
```


## What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

Aggregate the steps per interval (i.e. 05, 10, 15...) and also calculates the mean for each interval. This calculated mean value is then plotted as a time series.

```r
steps.interval <- aggregate(steps ~ interval, data=activity, FUN=mean)
plot(steps.interval, type="l",xlab="interval [in 5min]", ylab="Average daily activity pattern of steps",  main="average number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 


2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
steps.interval$interval[which.max(steps.interval$steps)]
```

```
## [1] 835
```


## Inputing missing values

There are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
sum(is.na(activity))
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

Since it is already calculated and represented as another variable, I will use the mean of steps for the 05 minute interval.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

Using the strategy described in step 2 above, will create the new data frame **activity.merged** which will replace all NA (from the steps column) with the mean of steps for the 05 minute interval.

```r
activity.merged = merge(activity, steps.interval, by="interval")
activity.merged$steps.x[is.na(activity.merged$steps.x)] = activity.merged$steps.y[is.na(activity.merged$steps.x)]
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

To be able to plot the histogram it is necessary to recalculate the aggregation of steps.

```r
activity.merged <- aggregate(steps.x~interval,activity.merged,sum)
hist(activity.merged$steps.x, xlab="Total steps by day", ylab="Frequency [Days]",main="Histogram : Number of daily steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png) 

Now, recalculate the mean and median total number os steps taken per day.

```r
mean2 <- mean(activity.merged$steps, na.rm=TRUE)
median2 <- median(activity.merged$steps, na.rm=TRUE)
```


```
## [1] 2280.339
```

```
## [1] 2080.906
```

***Analysis:*** The histogram now has many more acumulated values under 2000 steps a day - even the shape of the new histogram has changed. The values of mean and median also follow this pattern since they went from more than 10000 to less than 3000. Replacing the NA values for the steps by a very low value, completely changed the scenario.

## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
daytype <- function(date) {
    if (weekdays(as.Date(date)) %in% c("Saturday", "Sunday")) {
        "weekend"
    } else {
        "weekday"
    }
}
activity$daytype <- as.factor(sapply(activity$date, daytype))
str(activity)
```

```
## 'data.frame':	17568 obs. of  4 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
##  $ daytype : Factor w/ 2 levels "weekday","weekend": 1 1 1 1 1 1 1 1 1 1 ...
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
par(mfrow=c(2,1))
for (type in c("weekend", "weekday")) {
    steps.type <- aggregate(steps ~ interval,
                            data=activity,
                            subset=activity$daytype==type,
                            FUN=mean)
    plot(steps.type, type="l", main=type)
}
```

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png) 


