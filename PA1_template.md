# Reproducible Research: Peer Assessment 1

## Loading and preprocessing the data

In this exercice we will use data correponding to activy monitoring. 

First of alll we need to load data into workspace. As the data are in csv format we well use the *read.csv* function. The file contains three columns: steps (every 5 minutes interval), date (in YYYY-MM-DD fomat) and interval (an identifier of each interval)


```r
data=read.csv("activity.csv")
```
As a result we have a dataframe with three variables: steps, date an interval.

## What is mean total number of steps taken per day?

An option to know the number of steps per day is to build an histogram. First of all we used the function *aggregate* to summ all steps of each day(date). We stored the result in a new dataframe we called *stepsPerDay* (with tho columns, date and steps). After that we painted the histogram using the function *hist*.


```r
stepsPerDay=aggregate(steps ~ date, data, sum)
hist(stepsPerDay$steps,main="Steps per day")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png) 

In order to know the mean an median number os steps per day we use the R functions *mean* and *median*


```r
mean(stepsPerDay$steps)
```

```
## [1] 10766.19
```

```r
median(stepsPerDay$steps)
```

```
## [1] 10765
```
 
## What is the average daily activity pattern?

What about the daily activity pattern? when is more active this person. We ned to know the mean number os steps in each 5-minutes time interval. For calculating that we used again the *aggregate* R function but now we use the *interval* field as a factor. After that we can see the result in a plot. 


```r
stepsPerInterval=aggregate(steps ~ interval,data, mean)
plot(stepsPerInterval$steps,type='l',main='mean steps in each 5-minutes interval')
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

As we can see in this plot there is a zone of maximun activity around the 100 interval (in the morning, between 8 and 9). But when exactly this maximun is reached. We used the *which.max* R function to calculate that.


```r
which.max(stepsPerInterval$steps)
```

```
## [1] 104
```

## Imputing missing values

In some intervals the recorded number of steps is "NA", "Not a number". How many NA we have? A simple *summary* function give us this information. Another option, see below:


```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

We decided to build a new dataset replacing all NA values with the mean value of this 5-minutes interval. For doing that we used this code,


```r
data2=data
for (i in 1:length(data2$steps)){
  if(is.na(data2$steps[i])){
    data2$steps[i]=stepsPerInterval$steps[stepsPerInterval$interval == data2$interval[i]]
  }
}
```

In order to check if all NA values were replaced we count againg the NA. The result must be 0.

```r
sum(is.na(data2$steps))
```

```
## [1] 0
```

The new histogram of the steps taken each day is the following:


```r
stepsPerDay2=aggregate(steps ~ date, data2, sum)
hist(stepsPerDay2$steps,main="Steps per day")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png) 

And the new mean and median values are:


```r
mean(stepsPerDay2$steps)
```

```
## [1] 10766.19
```

```r
median(stepsPerDay2$steps)
```

```
## [1] 10766.19
```

As it can be seen, comparing with values calculated previously, the mean is the same an the median changes a little. It's due to the fact that all new values are equal to the mean value. In relation with the histogram, it's similar to the previous one, except the central bin that increased, as it's expected because all new values corresponds to this central interval.

## Are there differences in activity patterns between weekdays and weekends?

Finally we need to compare the activity on weekdays an on weekend. First of all we added a new column to the dataset, called *weekday* whith values *weekday* or *weekend*. To obtain this values we used the function *weekdays()* over the values of the corresponding *date* column. As the function weekdays depends on the language my implementation is suitable only for spanish or english language.


```r
data2$weekday <- lapply(as.Date(data2$date), weekdays)
data2$weekday <- lapply(data2$weekday, function(x) if (x == "Sábado" || x == "Domingo" || x == "Saturday" || x == "Sunday") {"weekend"} else {"weekday"})
```

For painting the pattern of activity on weekdays an on weekend we used a panel plot. The upper plot corresponds to the weekday activity and the botton plot to the weekend activity. The coding for painting these plot  is the following.


```r
WeekdayActivity=aggregate(steps ~ interval,subset(data2, weekday=="weekday"), mean)
WeekendActivity=aggregate(steps ~ interval,subset(data2, weekday=="weekend"), mean)
par(mfrow =c(2,1))
plot(WeekdayActivity$steps, type='l',ylim=c(0,250),main="Weekday activity", xlab="interval",ylab="steps")
plot(WeekendActivity$steps, type='l',ylim=c(0,250),main="Weekend activity", xlab="interval",ylab="steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png) 

As it can be seen the pattern of weekdays has a peak arround 100 interval that is not present on weekends. Perhaps this persons walks all mornings to his workplace!
