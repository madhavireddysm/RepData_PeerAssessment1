--- 
title: "PA1_template" 
output: 
  html_document: 
    keep_md: true 
---

```r
# Download and unzip data file.

echo = TRUE
setwd("/Users/MadhaviReddy/Reproducible Research/Assignment_1")
downloadURL <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
downloadFile <- "activity.zip"
if(!(file.exists("activity.csv")))
{  
    if(!file.exists(downloadFile)) 
    {
      download.file(downloadURL, downloadFile)
    }  
    unzip(downloadFile, overwrite = T)
}  

# Loading and preprocessing the data
# 1. Load the data (i.e. 𝚛𝚎𝚊𝚍.𝚌𝚜𝚟())
# 2. Process/transform the data (if necessary) into a format suitable for your analysis.

echo = TRUE
activity <- read.csv("activity.csv", sep=",", header=T)
activity$date <- as.Date(activity$date, "%Y-%m-%d")
str(activity)

# What is mean total number of steps taken per day?
# 1. Calculate the total number of steps taken per day
# 2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day
# 3. Calculate and report the mean and median of the total number of steps taken per day

echo = TRUE
TotalSteps <- tapply(activity$steps, activity$date, sum, na.rm=T)
echo = TRUE
hist(TotalSteps, col = "green", breaks=10, main="Histogram of the total number of steps taken each day", xlab="Total steps per day", ylab = "Frequency")
echo = TRUE
Mean_Steps <- mean(TotalSteps)
Median_Steps <- median(TotalSteps)
print(c("Mean: ", Mean_Steps))
print(c("Median: ", Median_Steps))

# What is the average daily activity pattern?
# 1. Make a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
# 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

echo = TRUE
AvgSteps <- aggregate(steps ~ interval, data = activity, mean, na.rm = TRUE)
echo = TRUE
plot(steps ~ interval, data = AvgSteps, type="l", xlab = "5-min interval", ylab = "Average numbe of steps taken per day")
echo = TRUE
MaxStepInterval <- AvgSteps[which.max(AvgSteps$steps),"interval"]
print(c("5-min interval that contains maximum number of steps is", MaxStepInterval))

# Inputing missing values
# 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with 𝙽𝙰s)
# 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
# 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
# 4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

echo = TRUE
missing_values <- sum(!complete.cases(activity))
print(c("Total number of missing values", missing_values ))

echo = TRUE
AvgStepsPerInterval <- function(interval)
{
    AvgSteps[AvgSteps$interval==interval,"steps"]
}
activity_NA <- activity  
flag = 0
for (i in 1:nrow(activity_NA)) {
    if (is.na(activity_NA[i,"steps"])) {
        activity_NA[i,"steps"] <- AvgStepsPerInterval(activity_NA[i,"interval"])
        flag = flag + 1
    }
}
head(activity)
head(activity_NA)
missing_values1 <- sum(!complete.cases(activity_NA))
print(c("Total number of missing values", missing_values1 ))

echo = TRUE
TotalStepsPerDay <- aggregate(steps ~ date, data = activity_NA, sum)
hist(TotalStepsPerDay$steps, col = "green", xlab = "Total Number of Steps", 
     ylab = "Frequency", main = "Histogram of Total Number of Steps taken each Day")

MeanSteps <- mean(TotalStepsPerDay$steps)
MedianSteps <- median(TotalStepsPerDay$steps)

Difference <- rbind( data.frame(mean = c(Mean_Steps, MeanSteps), median = c(Median_Steps, MedianSteps)))
rownames(Difference) <- c("with NA's", "without NA's")
print(Difference)
print("Median value is the same as the value before inputing missing data, but the mean value has changed. Replacing NA's with average steps taken per interval affected Mean values.")

# Are there differences in activity patterns between weekdays and weekends?
# 1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
# 2. Make a panel plot containing a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

echo = TRUE
activity_NA$weekday <- c("weekday")
activity_NA[weekdays(as.Date(activity_NA[, 2])) %in% c("Saturday", "Sunday", "samedi", "dimanche", "saturday", "sunday", "Samedi", "Dimanche"), ][4] <- c("weekend")
table(activity_NA$weekday == "weekend")
activity_NA$weekday <- factor(activity_NA$weekday)

echo = TRUE
activity_NA_weekend <- subset(activity_NA, activity_NA$weekday == "weekend")
activity_NA_weekday <- subset(activity_NA, activity_NA$weekday == "weekday")
mean_activity_NA_weekday <- tapply(activity_NA_weekday$steps, activity_NA_weekday$interval, mean)
mean_activity_NA_weekend <- tapply(activity_NA_weekend$steps, activity_NA_weekend$interval, mean)

echo = TRUE
library(lattice)
weekday <- NULL
weekend <- NULL
final <- NULL
weekday <- data.frame(interval = unique(activity_NA_weekday$interval), avg = as.numeric(mean_activity_NA_weekday), day = rep("weekday", length(mean_activity_NA_weekday)))
weekend <- data.frame(interval = unique(activity_NA_weekend$interval), avg = as.numeric(mean_activity_NA_weekend), day = rep("weekend", length(mean_activity_NA_weekend)))
final <- rbind(weekday, weekend)
xyplot(avg ~ interval | day, data = final, layout = c(1, 2), 
       type = "l", ylab = "Number of steps")
```       
