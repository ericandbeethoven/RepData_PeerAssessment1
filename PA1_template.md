# Reproducible Research: Peer Assessment 1
## Libraries Required

```r
library(Hmisc)
```

```
## Loading required package: grid
## Loading required package: lattice
## Loading required package: survival
## Loading required package: Formula
## Loading required package: ggplot2
## 
## Attaching package: 'Hmisc'
## 
## The following objects are masked from 'package:base':
## 
##     format.pval, round.POSIXt, trunc.POSIXt, units
```

```r
library(ggplot2)
library(scales)
```


## Loading and preprocessing the data

```r
# Download and unzip data
fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
if (!file.exists("./activity.zip")){
  download.file(fileUrl,destfile="./activity.zip")
}
unzip(zipfile="./activity.zip",exdir="./")

# Build Dataframe from unzipped data
activity.df = read.csv("./activity.csv", header = TRUE, sep = ",")

# Preprocess the data
activity.df = transform(activity.df,date=as.Date(date))
time = formatC(activity.df$interval/100, 2, format = "f")
activity.df$date.time = as.POSIXct(paste(activity.df$date, time), 
                                    format = "%Y-%m-%d %H.%M", 
                                     tz = "GMT")
activity.df$time = format(activity.df$date.time, 
                          format = "%H:%M:%S")
activity.df$time = as.POSIXct(activity.df$time, 
                              format = "%H:%M:%S")
```


## What is mean total number of steps taken per day?

### 1. Calculate the total number of steps taken per day

```r
x = tapply(activity.df$steps, activity.df$date, sum, na.rm=T) 
```

### 2. Make a histogram of the total number of steps taken each day

```r
hist(x, 
     main="Histogram for Activity - Sum of Steps per Day by Frequency", 
     xlab="Total Steps per Day", 
     ylab = "Frequency",
     border="blue", 
     col="green",
     ylim = c(0,20),
     las=1,
     breaks=c(0, seq(2500, 25000, 2500))
     )
```

![](figure-html/unnamed-chunk-4-1.png) 

### 3. Calculate and report the mean and median of the total number of steps taken per day

```r
mean(x, na.rm=T)
```

```
## [1] 9354.23
```

```r
median(x, na.rm=T)
```

```
## [1] 10395
```

## What is the average daily activity pattern?

### 1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
x = tapply(activity.df$steps, activity.df$time, mean, na.rm = T) 
activity.plot = data.frame(time = as.POSIXct(names(x)), steps=x)

plot(steps~time,
     data=activity.plot,
     type="l",
     main = "Time Series for Activity - Mean No. of Steps by 5 min Interval",
     sub = "Across Date Range From 10/1/12 - 11/30/12",
     xlab = "Time of Day",
     ylab = "Mean No. of Steps",
     col ="blue",
     lwd = 1,
     lty = 1
    )
```

![](figure-html/unnamed-chunk-6-1.png) 

### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
which.max(activity.plot$steps)
```

```
## 2015-10-17 08:35:00 
##                 104
```

## Imputing missing values
### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(activity.df))
```

```
## [1] 2304
```

### 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
activity.df.imputed = activity.df
activity.df.imputed$steps = with(activity.df.imputed, impute(steps, mean))
```

### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
activity.df.imputed = transform(activity.df.imputed,date=as.Date(date))
```

### 4. Make a histogram of the total number of steps taken each day. Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
x = tapply(activity.df.imputed$steps, activity.df.imputed$date, sum)

hist(x, 
     main="Histogram for Activity - Sum of Steps per Day by Frequency", 
     sub = "Contains Imputed Values for NAs",
     xlab="Total Steps per Day", 
     ylab = "Frequency",
     border="blue", 
     col="gray",
     ylim = c(0,30),
     las=1,
     breaks=c(0, seq(2500, 25000, 2500))
)
```

![](figure-html/unnamed-chunk-11-1.png) 

```r
mean(x)
```

```
## [1] 10766.19
```

```r
median(x)
```

```
## [1] 10766.19
```

As is shown, imputing the NA values has increased both the mean and median.

## Are there differences in activity patterns between weekdays and weekends?
There are differences in activity patterns between weekdays and weekends. There is more activity in the morning during the weekdays.There is more activity in the late morning and afternoon during the weekends. There is more activity in the evening during the weekends. There is similar activity during from 22:00 - 05:00 on weekends and weekdays.

### 1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
WeekendORWeekday <- function(date) {
  if (weekdays(date) %in% c("Saturday", "Sunday")) {
    return("weekend")
  } else {
    return("weekday")
  }
}

activity.df.imputed$IsWeekend = as.factor(sapply(activity.df.imputed$date.time, WeekendORWeekday))
```

### 2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis)

```r
# subset into weekend and weekday
activity.df.imputed.weekday = subset(activity.df.imputed, activity.df.imputed$IsWeekend == "weekday")
activity.df.imputed.weekend = subset(activity.df.imputed, activity.df.imputed$IsWeekend == "weekend")

# calculate mean for weekend / weekday and build row combined dataframe
x1 = tapply(activity.df.imputed.weekday$steps, activity.df.imputed.weekday$time, mean) 
x2 = tapply(activity.df.imputed.weekend$steps, activity.df.imputed.weekend$time, mean)
activity.plot1 = data.frame(time = as.POSIXct(names(x1)), steps=x1)
activity.plot1$IsWeekend = "weekday"
activity.plot2 = data.frame(time = as.POSIXct(names(x2)), steps=x2)
activity.plot2$IsWeekend = "weekend"
activity.plot = rbind(activity.plot1,activity.plot2)

# extract time of 'date+time' (POSIXct) in hours as numeric
dtTime = as.numeric(activity.plot$time - trunc(activity.plot$time, "days"))/ 3600
class(dtTime) = "POSIXct"

# Make plot
p = ggplot(activity.plot,aes(x=dtTime, y=steps, color = IsWeekend))

# Add plot line
p = p + geom_line(shape = 21, size = 1, col="blue") 

# Add facet by type
p = p + facet_grid(IsWeekend ~ .)

# Add plot title & labels
p = p + ggtitle("Time Series - Mean Steps by 5-minute interval") + 
    ylab("Mean No. of Steps") +
    xlab("Time of Day")

# Scale & format x axis
p = p + scale_x_datetime(labels = date_format("%S:00")) 

# White background and black grid lines
p <- p + theme_bw()

print(p)  
```

![](figure-html/unnamed-chunk-13-1.png) 




