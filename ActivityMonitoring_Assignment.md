---
title: "Reproducible Research Coursera"
output:
  
  html_document: 
    cache: yes
    fig_caption: yes
    keep_md: yes
---



### Retrieving the data & loading approprirate packages
The first step we have to do is to download the data, which is in a zip file. We will download the file and name it temp
Then once we have downloaded it, will unzip the zip file and read the activity.csv file
The last step of the import proccess is to assign the csv data to a data
For this project, we need the following packages:
*dplyr


```r
temp <- tempfile()
download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip",temp)
data <- read.csv(unz(temp, "activity.csv"))
unlink(temp)
data$date <- as.Date(data$date)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(ggplot2)
```
#What is mean total number of steps taken per day?

#### Calculate the total number of steps taken per day
Using the dplyr package, we can use the summarise function alog with the group_by function. This allows us to sum by the grouped values

```r
tot_steps_per_day<- data.frame(data %>%
  group_by(date) %>%
summarise(sum1 = sum(steps, na.rm = TRUE)))
```
#### Make a histogram of the total number of steps taken each day

To answer this questions we have to group the steps by day and sum them. This can be accomplished with the dplyr package.

Once have have calculated the totals per day, we use the base plotting system to plot the histogram. 

```r
data %>%
  group_by(date) %>%
summarise(sum1 = sum(steps, na.rm = TRUE)) %>%
with(hist(sum1, breaks = 15, xlab = "Total Number of Steps Taken Each Day", main ="Total Number of Steps Taken Each Day Frequency"))
```

![](ActivityMonitoring_Assignment_files/figure-html/data-1.png)<!-- -->




#### Calculate and report the mean and median of the total number of steps taken per day

What is mean total number of steps taken per day?

```r
data %>%
  group_by(date) %>%
summarise(sum1 = sum(steps, na.rm = TRUE)) %>%
with(mean(sum1, na.rm = TRUE))
```

```
## [1] 9354.23
```
#### What is median total number of steps taken per day?

```r
data %>%
  group_by(date) %>%
summarise(sum1 = sum(steps, na.rm = TRUE)) %>%
with(median(sum1, na.rm = TRUE))
```

```
## [1] 10395
```


# What is the average daily activity pattern?
#### Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
Let's make a line chart illustrating the average number of steps taken for each 5 minute interval


```r
data %>%
  group_by(interval) %>%
  summarise(avg1 = mean(steps, na.rm = TRUE)) %>%
  with(plot(interval, avg1, type ="l", ylab ="Average number of steps taken"))
```

![](ActivityMonitoring_Assignment_files/figure-html/data3-1.png)<!-- -->




#### Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
First lets create a dataframe with the averages
Then lets subset it down to only look at the record with the max. This should correspond to the peak of the line chart


```r
df1 <- 
  data.frame(data %>%
  group_by(interval) %>%
  summarise(avg1 = mean(steps, na.rm = TRUE)))
  filter(df1, avg1 == max(avg1), interval)
```

```
##   interval     avg1
## 1      835 206.1698
```

# Imputing missing values

####  Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

We will use the is.na function which returns a boolean value. If we sum a boolean, we get the count of the missing values


```r
  sum(is.na(data$steps))
```

```
## [1] 2304
```


# Devise a strategy for filling in all of the missing values in the dataset.
####  Create a new dataset that is equal to the original dataset but with the missing data filled in.

We are going to replace the missing values with the means by time interval. 
We are calling the new dataset df2


```r
 impute.mean <- function(x) replace(x, is.na(x), mean(x, na.rm = TRUE))
df2 <-
  data.frame(data %>%
   group_by(interval) %>%
   mutate(
     steps = impute.mean(steps),
))
```

#### Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day

```r
df2 %>%
  group_by(date) %>%
summarise(sum2 = sum(steps, na.rm = TRUE)) %>%
with(hist(sum2, breaks = 15, xlab = "Total Number of Steps Taken Each Day", main ="Total Number of Steps Taken Each Day Frequency"))
```

![](ActivityMonitoring_Assignment_files/figure-html/data 7-1.png)<!-- -->

#### Do these values differ from the estimates from the first part of the assignment?
It moves the values in the left most bar to the middle

### What is the impact of imputing missing data on the estimates of the total daily number of steps?
Lets first start by bringing up the old

```r
tot_steps_per_day<- data.frame(data %>%
  group_by(date) %>%
summarise(sum1 = sum(steps, na.rm = TRUE)))
summary(tot_steps_per_day)
```

```
##       date                 sum1      
##  Min.   :2012-10-01   Min.   :    0  
##  1st Qu.:2012-10-16   1st Qu.: 6778  
##  Median :2012-10-31   Median :10395  
##  Mean   :2012-10-31   Mean   : 9354  
##  3rd Qu.:2012-11-15   3rd Qu.:12811  
##  Max.   :2012-11-30   Max.   :21194
```
Lets compare to the new

```r
tot_steps_per_day2<- data.frame(df2 %>%
  group_by(date) %>%
summarise(sum2 = sum(steps, na.rm = TRUE)))
summary(tot_steps_per_day2)
```

```
##       date                 sum2      
##  Min.   :2012-10-01   Min.   :   41  
##  1st Qu.:2012-10-16   1st Qu.: 9819  
##  Median :2012-10-31   Median :10766  
##  Mean   :2012-10-31   Mean   :10766  
##  3rd Qu.:2012-11-15   3rd Qu.:12811  
##  Max.   :2012-11-30   Max.   :21194
```
The mean increased a lot while the median increased slightly less. Becayse we used the mean, our median and mean are now exactly the same

#Are there differences in activity patterns between weekdays and weekends?
#### Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
df4 <- df2
df4$date <- as.Date(df4$date)
wkds <- weekdays(df4$date)
wkds[wkds %in% c("Saturday","Sunday")] <- "Weekend"
wkds[wkds != "Weekend"] <-"Weekday"
df4<-cbind(df4,wkds)
```



```r
df6 <- aggregate( steps ~ interval + wkds,df2, mean)
ggplot(df6, aes(interval, steps) )+geom_line()+facet_wrap(~wkds, ncol = 1)
```

![](ActivityMonitoring_Assignment_files/figure-html/data10-1.png)<!-- -->
