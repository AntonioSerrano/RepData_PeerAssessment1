# Reproducible Research: Peer Assessment 1
Antonio Serrano  
March 13, 2016  

### Introduction

This document has been written to complete the assignment "Course Project 1" in the context of the Reproducible Research course offered by Johns Hopkins University through Coursera.org

### Basic settings


```r
echo = TRUE  # To make code always visible

options(scipen = 1)  # Turn off scientific notations for numbers
```

### Loading and processing data

1. Load the data (i.e. read.csv())


```r
# Downloading the zip file:

if(!file.exists('activity.zip')){
    download.file(url = "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", destfile= "./activity.zip", method = "curl")
}

# Unzipping file:

if(!file.exists('activity.csv')){
    unzip('activity.zip')
}

# Reading data set using the "read.table" function:

df <- read.table(file = "./activity.csv", header = TRUE, sep = ",", na.strings = "NA", colClasses = c("integer", "Date", "integer"))

# Checking data set dimensions:

dim(df) # 17568 rows x 3 columns
```

```
## [1] 17568     3
```

2. Process/transform the data (if necessary) into a format suitable for your analysis

It is not necessay to make any transformation.

### What is mean total number of steps taken per day?

1. Calculate the total number of steps taken per day


```r
# Note: for this part of the assignment, we will ignore the missing values in the data set.

# Calculating the total number of steps taken per day:

stepsPerDay <- aggregate(steps ~ date, data = df, sum, na.rm = TRUE) # data frame class
```

2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day


```r
# Loading ggplot2 package:

library(ggplot2)

# Drawing histogram of the total number of steps taken each day using geom_histogram in ggplot2:

ggplot(stepsPerDay, aes(steps)) +
        geom_histogram(binwidth = 1000, color = "dodgerblue4", fill = "dodgerblue") +
        ggtitle("Histogram of the total number of steps taken each day") +
        xlab("Total steps per day") +
        ylab("Frequency") +
        theme(plot.title = element_text(family = "Helvetica", size=14, face="bold"),
              axis.title.x = element_text(family = "Helvetica", size=12),
              axis.title.y = element_text(family = "Helvetica", size=12))
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)

3. Calculate and report the mean and median of the total number of steps taken per day


```r
# Calculating the mean and median of the aggregated steps per day:

stepsPerDayMean <- mean(stepsPerDay$steps)
print(paste("The mean steps per day is: ", stepsPerDayMean))
```

```
## [1] "The mean steps per day is:  10766.1886792453"
```

```r
stepsPerDayMedian <- median(stepsPerDay$steps)
print(paste("The median steps per day is: ", stepsPerDayMedian))
```

```
## [1] "The median steps per day is:  10765"
```

### What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis):


```r
# Aggregating data on steps by interval and calculating the mean:

averageStepsPerInterval <- aggregate(steps ~ interval, data = df, mean, na.rm=TRUE)

# Drawing the time series plot:

ggplot(averageStepsPerInterval, aes(interval, steps)) +
        geom_line(color = "goldenrod", size = 0.8) +
        ggtitle("Average steps per five minute interval") +
        xlab("Interval number") +
        ylab("Average number of steps taken") +
        theme(plot.title = element_text(family = "Helvetica", size=14, face="bold"),
              axis.title.x = element_text(family = "Helvetica", size=12),
              axis.title.y = element_text(family = "Helvetica", size=12))
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
# Interval containing the maximum number of steps:

maxInterval <- averageStepsPerInterval[which.max(averageStepsPerInterval$steps),1]

# Number of step taken in that interval:

maxSteps <- round(max(averageStepsPerInterval$steps), 0)

# Printing the answer:

print(paste("The 5-minute interval containing the maximum number of steps is interval no.", maxInterval, ". The maximum number of steps in that interval was", maxSteps))
```

```
## [1] "The 5-minute interval containing the maximum number of steps is interval no. 835 . The maximum number of steps in that interval was 206"
```

### Imputing missing values

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs:


```r
# Counting missing values in the data set:

missing <- sum(is.na(df$steps))

# Percentage of missing values over the total data points:

missingPercent <- round((missing/dim(df)[1])*100, 2)

print(paste("There are", missing, "missing values. That is, a", missingPercent, "% over the total data points"))
```

```
## [1] "There are 2304 missing values. That is, a 13.11 % over the total data points"
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

We would use multiple imputation as strategy for filling in the missing values. Multiple imputation is normally considered a mathematically advanced approach, but R packages like MICE (Multiple Imputation by Chained Equation) makes the process very easy. Let's check it out.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in:


```r
# Loading "mice" package:

library("mice")

# Keeping only steps and interval columns:

df2 <- df[c("steps","interval")]

# Imputing missing values using "mice" basic function:

dfImputed <- mice(data = df2, seed = 1000)
```

```
## 
##  iter imp variable
##   1   1  steps
##   1   2  steps
##   1   3  steps
##   1   4  steps
##   1   5  steps
##   2   1  steps
##   2   2  steps
##   2   3  steps
##   2   4  steps
##   2   5  steps
##   3   1  steps
##   3   2  steps
##   3   3  steps
##   3   4  steps
##   3   5  steps
##   4   1  steps
##   4   2  steps
##   4   3  steps
##   4   4  steps
##   4   5  steps
##   5   1  steps
##   5   2  steps
##   5   3  steps
##   5   4  steps
##   5   5  steps
```

```r
# Note1: since we did not provide a specific method of imputation, "mice"" applies Predictive Mean Matching (PMM) method by default (method = pmm):

dfImputed$meth
```

```
##    steps interval 
##    "pmm"       ""
```

```r
# Note2: in addition, we did not specify the number of multiple imputations. In this case, the default value is m = 5.

# If we wanted to check the imputed data for the variable "steps", we would do this:

# dfImputed$imp$steps

# Getting back the completed dataset with the imputed missing values:

dfImputed <- complete(dfImputed)
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
# Calculating the total number of steps taken per day:

stepsPerDay2 <- aggregate(steps ~ date, data = dfImputed, sum, na.rm = TRUE) # data frame class

# Drawing histogram of the total number of steps taken each day:

ggplot(stepsPerDay2, aes(steps)) +
        geom_histogram(binwidth = 1000, color = "firebrick4", fill = "firebrick2") +
        ggtitle("Histogram of the total number of steps per day imputating missing values") +
        xlab("Total steps per day") +
        ylab("Frequency") +
        theme(plot.title = element_text(family = "Helvetica", size=14, face="bold"),
              axis.title.x = element_text(family = "Helvetica", size=12),
              axis.title.y = element_text(family = "Helvetica", size=12))
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)

```r
# At this point, we could compare the two histograms (with and without imputed missing values) as follows:

# Creating "multiplot" function:

# ggplot objects can be passed in ..., or to plotlist (as a list of ggplot objects)
# - cols:   Number of columns in layout
# - layout: A matrix specifying the layout. If present, 'cols' is ignored.

# If the layout is something like matrix(c(1,2,3,3), nrow=2, byrow=TRUE),
# then plot 1 will go in the upper left, 2 will go in the upper right, and
# 3 will go all the way across the bottom.

multiplot <- function(..., plotlist=NULL, file, cols=1, layout=NULL) {
  library(grid)

  # Make a list from the ... arguments and plotlist
  plots <- c(list(...), plotlist)

  numPlots = length(plots)

  # If layout is NULL, then use 'cols' to determine layout
  if (is.null(layout)) {
    # Make the panel
    # ncol: Number of columns of plots
    # nrow: Number of rows needed, calculated from # of cols
    layout <- matrix(seq(1, cols * ceiling(numPlots/cols)),
                    ncol = cols, nrow = ceiling(numPlots/cols))
  }

 if (numPlots==1) {
    print(plots[[1]])

  } else {
    # Set up the page
    grid.newpage()
    pushViewport(viewport(layout = grid.layout(nrow(layout), ncol(layout))))

    # Make each plot, in the correct location
    for (i in 1:numPlots) {
      # Get the i,j matrix positions of the regions that contain this subplot
      matchidx <- as.data.frame(which(layout == i, arr.ind = TRUE))

      print(plots[[i]], vp = viewport(layout.pos.row = matchidx$row,
                                      layout.pos.col = matchidx$col))
    }
  }
}

# Storing the two histograms as "hist1" and "hist2" respectively:

hist1 <- ggplot(stepsPerDay, aes(steps)) +
        geom_histogram(binwidth = 1000, color = "dodgerblue4", fill = "dodgerblue") +
        ggtitle("Histogram without imputed data") +
        xlab("Total steps per day") +
        ylab("Frequency") +
        theme(plot.title = element_text(family = "Helvetica", size=14, face="bold"),
              axis.title.x = element_text(family = "Helvetica", size=12),
              axis.title.y = element_text(family = "Helvetica", size=12))

hist2 <- ggplot(stepsPerDay2, aes(steps)) +
        geom_histogram(binwidth = 1000, color = "firebrick4", fill = "firebrick2") +
        ggtitle("Histogram with imputed data") +
        xlab("Total steps per day") +
        ylab("Frequency") +
        theme(plot.title = element_text(family = "Helvetica", size=14, face="bold"),
              axis.title.x = element_text(family = "Helvetica", size=12),
              axis.title.y = element_text(family = "Helvetica", size=12))

# Calling multiplot function:

multiplot(hist1, hist2, cols = 2)
```

![](PA1_template_files/figure-html/unnamed-chunk-10-2.png)

```r
# As we can see in the previous plot, the results are almost imperceptible.

# Calculating the mean and median of the aggregated steps per day:

stepsPerDayMean2 <- mean(stepsPerDay2$steps)
print(paste("The mean steps per day is: ", stepsPerDayMean2))
```

```
## [1] "The mean steps per day is:  10706.0327868852"
```

```r
stepsPerDayMedian2 <- median(stepsPerDay2$steps)
print(paste("The median steps per day is: ", stepsPerDayMedian2))
```

```
## [1] "The median steps per day is:  10765"
```

```r
# Therefore, the mean decreases from 10766 (when omitting missing values) to 10706 (when imputing them using pmm). The median does not change at all. So, the overall impact of imputing missing values is virtually unnoticeable in this case.
```

### Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
# Adding "date" column from df to dfImputed:

dfImputed$date <- df$date

# Adding "dayname" column to dfImputed data frame to specify the day name of each date:

dfImputed$dayname <- weekdays(dfImputed$date)

# Adding "dayType" as a factor variable to dfImputed data frame to differentiate between weekdays and weekend:

dfImputed$dayType <- as.factor(ifelse(dfImputed$dayname == "Saturday" |
                                dfImputed$dayname == "Sunday", "weekend", "weekdays"))
```

2. Make a panel plot containing a time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
# Drawing panel plot using ggplot:

# Aggregating data on steps by interval and calculating the mean:

averageStepsPerInterval2 <- aggregate(steps ~ interval + dayType, data = dfImputed, mean)

# Drawing the time series plot:

ggplot(averageStepsPerInterval2, aes(interval, steps, color = dayType)) +
        geom_line(size = 0.8) +
        ggtitle("Average steps per five minute interval on weekends/weekdays") +
        xlab("Interval number") +
        ylab("Average number of steps taken") +
        theme(plot.title = element_text(family = "Helvetica", size=14, face="bold"),
              axis.title.x = element_text(family = "Helvetica", size=12),
              axis.title.y = element_text(family = "Helvetica", size=12),
              legend.position = "none") +
        facet_wrap(~dayType, nrow = 2)
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)
