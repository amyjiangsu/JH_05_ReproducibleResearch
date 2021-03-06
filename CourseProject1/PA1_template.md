Reproducible Research - Peer Assessment 1
==========================================
Created by: Amy Su Jiang  
Date: November 14th, 2014  

### Loading and processing the data

__1. Load the data (i.e. read.csv())__
```{r}
unzip("activity.zip")
data  <- read.csv("activity.csv", colClasses = c("integer", "Date", "factor"))
```
  
__2. Process/transform the data (if necessary) into a format suitable for your analysis__
```{r}
data_noNA  <- na.omit(data)
```

### What is mean total number of steps taken per day?  
For this part of the assignment, you can ignore the missing values in the dataset.  
  
__1. Make a histogram of the total number of steps taken each day__
```{r}
data_noNA$month  <- as.numeric(format(data_noNA$date,"%m"))
library(ggplot2)
histGraph <- ggplot(data_noNA, aes(date, steps))
histGraph <- histGraph + geom_bar(stat = "identity", color="light blue", fill="light blue", width = 0.7)
histGraph <- histGraph + facet_grid(.~month, scales = "free")
histGraph <- histGraph + labs(title="Histograph of Total Number of Steps Taken Each Day\n")
print(histGraph)
```
<img src="image1.png">  
  
__2. Calculate and report the mean and median total number of steps taken per day__
The mean of total number of steps taken per day is:  
```{r}
mean(aggregate(data_noNA$steps, list(Date = data_noNA$date), FUN = "sum")$x)
```
  
The median of total number of steps taken per day is:  
```{r}
median(aggregate(data_noNA$steps, list(Date = data_noNA$date), FUN = "sum")$x)
```
  
### What is the average daily activity pattern?    
__1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)__
```{r}
avgSteps <- aggregate(data_noNA$steps, list(interval = as.numeric(as.character(data_noNA$interval))), FUN = "mean")
names(avgSteps)[2] <- "meanOfSteps"

plotGraph <- ggplot(avgSteps, aes(interval, meanOfSteps))
plotGraph <- plotGraph + geom_line(color = "light blue", size = 0.8)
plotGraph <- plotGraph + labs(title = "Time Series Plot of the 5-minute Interval\n")
print(plotGraph)
```
<img src="image2.png">

__2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?__ 
```{r}
avgSteps[avgSteps$meanOfSteps == max(avgSteps$meanOfSteps), ]
```

  
### Imputing missing values
__1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs__
```{r}
sum(is.na(data))
```
  
__2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.__   
_I will use the mean of 5-minute interval to replace each NA value in the steps column._
  
__3. Create a new dataset that is equal to the original dataset but with the missing data filled in.__
```{r}
data_new <- data 
for (i in 1:nrow(data_new)) {
    if (is.na(data_new$steps[i])) {
        data_new$steps[i] <- avgSteps[which(data_new$interval[i] == avgSteps$interval), ]$meanOfSteps
    }
}
```
  
__4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. __
```{r}
data_new$month  <- as.numeric(format(data_new$date,"%m"))
histGraph1 <- ggplot(data_new, aes(date, steps))
histGraph1 <- histGraph1 + geom_bar(stat = "identity", color="light blue", fill="light blue", width = 0.7)
histGraph1 <- histGraph1 + facet_grid(.~month, scales = "free")
histGraph1 <- histGraph1 + labs(title="Histograph of Total Number of Steps Taken Each Day \n(No Missing Values)\n")
print(histGraph1)
```
<img src="image3.png">  

__Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?__
```{r}
oldMean <- mean(aggregate(data_noNA$steps, list(Date = data_noNA$date), FUN = "sum")$x)
newMean  <- mean(aggregate(data_new$steps, list(Date = data_new$date), FUN = "sum")$x)
oldMedian <- median(aggregate(data_noNA$steps, list(Date = data_noNA$date), FUN = "sum")$x)
newMedian  <- median(aggregate(data_new$steps, list(Date = data_new$date), FUN = "sum")$x)
newMean - oldMean
newMedian - oldMedian
```    
_Hence the new mean of total steps taken per day is the same as that of the old mean, but the new median of total steps taken per day is greater than that of the old median._  

  
### Are there differences in activity patterns between weekdays and weekends?  
__1. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.__
```{r}
data_new$weekdays <- factor(format(data_new$date, "%A"))
levels(data_new$weekdays) <- list(weekday = c("Monday", "Tuesday",
                                             "Wednesday", 
                                             "Thursday", "Friday"),
                                 weekend = c("Saturday", "Sunday"))
```
  
__2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).__
```{r}
avgSteps <- aggregate(data_new$steps, 
                      list(interval = as.numeric(as.character(data_new$interval)), 
                           weekdays = data_new$weekdays),
                      FUN = "mean")
names(avgSteps)[3] <- "meanOfSteps"
library(lattice)
xyplot(avgSteps$meanOfSteps ~ avgSteps$interval | avgSteps$weekdays,
       layout = c(1, 2), type = "l", 
       xlab = "Interval", ylab = "Number of steps")
```  
<img src="image4.png">  