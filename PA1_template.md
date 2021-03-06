<span style="color:blue">Assignment 1 : Personal activity monitoring device</span>
========================================================

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The **data for this assignment** can be downloaded from the course web site:
[Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)

The variables included in this dataset are:
- steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)
- date: The date on which the measurement was taken in YYYY-MM-DD format
- interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

<span style="color:blue">Code for reading in the dataset and/or processing the data</span>
--------------------------------------------------


```r
# setwd("/Users/emmanuel/./R/data")
# if (!file.exists("activity.zip")) {
#         download.file(url="https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", destfile="activity.zip", method="libcurl")
#         unzip("activity.zip")  
# }
unzip("activity.zip")
activity <- read.csv("activity.csv")
str(activity)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
head(activity)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

All data are there ! But with a lot of NA values displayed. We will try to fix that quickly.

**0. Preprocessing**


```r
## Split the data by date
split_steps <- split(activity$steps, activity$date)
## str(split_steps) - we can see that there are no observations for day 1, 8, 32, 35, 40, 41, 45, 61

## Sum the data by day
steps_by_day <- rowSums(do.call(rbind, split_steps), na.rm = TRUE)

## Convert the named num data type to data frame
steps_by_day_df <- data.frame(Date=names(steps_by_day), steps=steps_by_day, row.names=NULL)

## Remove the days with no data
steps_no_na <- steps_by_day_df[-c(1, 8, 32, 35, 40, 41, 45, 61), ] 
```


<span style="color:blue">What is mean total number of steps taken per day?</span>
--------------------------------------------------------------

**1. Make a histogram of the total number of steps taken each day**

Whereas a bar graph, (or a bar chart, as it is sometimes referred to) is a way of showing a comparison of values by displaying numeric variable x for each level of factor, a histogram is used to show values of recorded data elements that are grouped or categorized. 



```r
steps_by_day_hist <- hist(steps_no_na$steps, breaks = 20, 
main="Steps taken in a day",
xlab="Number of Steps", ylim=c(0,10))
```

![plot of chunk histogram Steps taken in a day](figure/histogram Steps taken in a day-1.png) 

**2. Calculate and report the mean and median of the total number of steps taken per day**


```r
mean1 <- mean(steps_no_na$steps)
median1 <- median(steps_no_na$steps)
cat("The mean of the total number of steps", mean1)
```

```
## The mean of the total number of steps 10766.19
```

```r
cat("The median of the total number of steps", median1)
```

```
## The median of the total number of steps 10765
```


<span style="color:blue">What is the average daily activity pattern?</span>
--------------------------------------------------

**3. Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)**

```r
ave_by_interval <- colMeans(do.call(rbind, split_steps), na.rm = TRUE)
minutes_in_day <- activity$interval[1:288]
ave_steps_by_interval <- data.frame("Time" = minutes_in_day, "Average_Steps" = ave_by_interval)
intervals <- strptime(sprintf("%04d", as.numeric(ave_steps_by_interval$Time)), format="%H%M")

plot(intervals, ave_steps_by_interval$Average_Steps, type="l", 
xlab= "Daytime", ylab= "Average Number of Steps", col="darkred" , lwd=1, main = "Average number of steps by 5 minute interval")
```

![plot of chunk Time series plot](figure/Time series plot-1.png) 


**4. Which 5-minute interval that, on average, contains the maximum number of steps?**


```r
max(ave_steps_by_interval$Average_Steps)
```

```
## [1] 206.1698
```

```r
ave_steps_by_interval$Time[which.max(ave_steps_by_interval$Average_Steps)]
```

```
## [1] 835
```

```r
cat("The 5 minute interval that contains the most steps is at", ave_steps_by_interval$Time[which.max(ave_steps_by_interval$Average_Steps)], "am", "with", max(ave_steps_by_interval$Average_Steps), "steps")
```

```
## The 5 minute interval that contains the most steps is at 835 am with 206.1698 steps
```


<span style="color:blue">Imputing missing data</span>
-------------------------------------------------

**5. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)**

```r
missingvalues <- sum(is.na(activity))
cat("The total number of missing values in the data set is", missingvalues)
```

```
## The total number of missing values in the data set is 2304
```


**6. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.**

My strategy is to use the mean for that 5-minute interval to fill each NA value in the steps column.


**7. Create a new dataset that is equal to the original dataset but with the missing data filled in**


```r
newData <- activity 
for (i in 1:nrow(newData)) {
    if (is.na(newData$steps[i])) {
        noNA <- na.omit(activity)
        avgSteps <- aggregate(noNA$steps, list(interval = as.numeric(as.character(noNA$interval))), FUN = "mean")
        names(avgSteps)[2] <- "meanOfSteps"
        newData$steps[i] <- avgSteps[which(newData$interval[i] == avgSteps$interval), ]$meanOfSteps
    }
}
```


```r
head(newData)
```

```
##       steps       date interval
## 1 1.7169811 2012-10-01        0
## 2 0.3396226 2012-10-01        5
## 3 0.1320755 2012-10-01       10
## 4 0.1509434 2012-10-01       15
## 5 0.0754717 2012-10-01       20
## 6 2.0943396 2012-10-01       25
```


**8. Make a histogram of the total number of steps taken each day**

```r
totalbyday<-tapply(newData$steps, newData$date, sum)
hist(totalbyday, main = "Total Number of Steps per Day", xlab="Total number of steps", ylim=c(0,25), breaks = 10, col = "69", font.lab=2, cex.main=1.5)
```

![plot of chunk histogram filled with NA values](figure/histogram filled with NA values-1.png) 

**9. Calculate and report the mean and median total number of steps taken per day**

What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
sum <- as.numeric(totalbyday)
mean2 <- mean(sum)
median2 <- median(sum)
diffmean <- mean2 - mean1
diffmedian <- median2 - median1
cat("The difference is very weak between first figures obtained and those ones with mean equal to", mean2, "and median similarly equal to", median2)
```

```
## The difference is very weak between first figures obtained and those ones with mean equal to 10766.19 and median similarly equal to 10766.19
```

```r
cat("The difference is equal to", diffmean, "between the means and equates", diffmedian, "for the medians")
```

```
## The difference is equal to 0 between the means and equates 1.188679 for the medians
```

Therefore, the impact of the missing data seems rather low, at least when estimating the total number of steps per day.


<span style="color:blue">Are there differences in activity patterns between weekdays and weekends?</span>
------------------------------------------------------------

**10. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.**


```r
Sys.setlocale("LC_TIME", "English")
```

```
## [1] "English_United States.1252"
```

```r
# weekdays(Sys.Date()+0:6)
dayType <- function(date) {
    if (weekdays(as.Date(date)) %in% c("Saturday", "Sunday")) {
        "weekend"
    } else {
        "weekday"
    }
}
newData$dayType <- as.factor(sapply(newData$date, dayType))

table(newData$dayType)
```

```
## 
## weekday weekend 
##   12960    4608
```

**11. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.**


```r
par(mfrow = c(2, 1))
library(lattice)
stepsByDay <- aggregate(steps ~ interval + dayType, data = newData, FUN = mean)
names(stepsByDay) <- c("interval", "daytype", "steps")
xyplot(steps ~ interval | daytype, stepsByDay, type = "l", layout = c(1, 2), xlab = "Interval", ylab = "Number of steps")
```

![plot of chunk Panel plot](figure/Panel plot-1.png) 

Obviously, during weekdays a peak occurs around 08:30 when people go to work. Also activity starts earlier than weekend as people tend to wake up later on weekend days.
