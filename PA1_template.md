# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data


```r
###Load Libraries###
library(dplyr)
library(ggplot2)

##READ IN CSV###
x <- read.csv("activity.csv", stringsAsFactors = FALSE)

###Convert Date variable to Date vector; replace in original dataframe and ensure columns are named correctly###
date <- strptime(x$date, "%Y-%m-%d")
df <-data.frame(x[[1]], date, x[[3]])
colnames(df) <- c("steps", "date", "interval")

###Create new variable called dow, populate with name of day###
df <- mutate(df, dow = weekdays(date))
                                
###Convert names of days into factor with Weekend and Weekday; add new variable to moriginal data frame###                             
weekdayslist <- c('Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday')
daytype <- factor((df$dow %in% weekdayslist), levels=c(FALSE, TRUE), labels=c('weekend', 'weekday'))
df <- data.frame(df, daytype)
```



## What is mean total number of steps taken per day?


```r
###Use DPLYR package to create data frame grouped by date###
stepsbyday<- df %>%
        group_by(date) %>%
        summarize(dailysteps = sum(steps, na.rm=TRUE))

stepsbyday <- filter(stepsbyday, dailysteps > 0)

meandailysteps <- as.integer(mean(stepsbyday$dailysteps, na.rm=TRUE))
mediandailysteps <- as.integer(median(stepsbyday$dailysteps, na.rm=TRUE))

hist(stepsbyday$dailysteps, 
     main = "Histogram of Daily Steps", xlab="Total Steps by Day", ylab="Number of Days", 
     col="lightblue")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png) 

Mean daily steps = 10766  
Median daily steps = 10765

## What is the average daily activity pattern?


```r
intervalsteps <- df %>%
        group_by(interval) %>%
        summarize(stepsinterval = sum(steps, na.rm=TRUE),  meanintervalsteps = mean(steps, na.rm=TRUE) )

moststeps <- top_n(intervalsteps, 1, meanintervalsteps)
moststeps <- moststeps[[1]]


plot(intervalsteps$interval, intervalsteps$meanintervalsteps, type="l",
     main = "Average Steps Per Interval Period", xlab="Interval Period", ylab="Mean Steps during Interval", 
     col="red")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

The 5-minute interval that on average across all the days in the dataset
contains the maximum number of steps is: 835


## Imputing missing values


```r
###Create new dataframe by replacing NA's with average number of steps of interval across all days### 
newdf <- mutate(df, steps = ifelse(is.na(steps) == "TRUE", intervalsteps$meanintervalsteps, steps))
                
                stepsbydayadj <- newdf %>%
                        group_by(date) %>%
                        summarize(dailysteps = sum(steps, na.rm=TRUE))

meandailystepsadj = as.integer(mean(stepsbydayadj$dailysteps, na.rm=TRUE))
mediandailystepsadj = as.integer(median(stepsbydayadj$dailysteps, na.rm=TRUE))

                
hist(stepsbydayadj$dailysteps, 
     main = "Histogram of Daily Steps", xlab="Total Steps by Day", ylab="Number of Days", 
     col="lightblue")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 


Imputed Mean: 10766  
Actual Mean: 10766  
Difference:  0

Imputed Median: 10766  
Actual Median: 10765  
Difference: 1

## Are there differences in activity patterns between weekdays and weekends?


```r
intervalstepsbydaytype <- df %>%
        group_by(daytype, interval) %>%
        summarize(meandailysteps = mean(steps, na.rm=TRUE)  )


qplot(interval, meandailysteps, data = intervalstepsbydaytype, facets = .~daytype) + 
        geom_line()+
        labs(list(title = "Steps by Interval Period -- Weekend vs Weekday", 
                  x = "Interval Period", y = "Total Steps"))
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 
   
Yes, there are differences between weekdays and weekends.  
- There appears to be more consistent activity throughout the day on the weekend.  
- There appears to be a period of spiked activity in the morning on weekdays, perhaps a walk to work, followed by minimal activity until about 5-6pm, perhaps when the subject leaves work.  