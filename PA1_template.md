---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document: PA1_template.html
  keep_md: true
---

### Loading and preprocessing the data


```r
    # Position root directory for analysis (hardwired)
    workdir <- "/Users/fbreschi/Documents/GitHubRepos/RepData_PeerAssessment1"
    setwd(workdir)    
    # (1) Read data (raw data)
    rdata <- read.csv("activity.csv")    
    # (2) Clean data
    cdata <- rdata[complete.cases(rdata),]
    # Includes
    library(ggplot2)
    # Commodity vectors    
    # Time span for raw data
    rv <-  unique(rdata$date)  
    # Time span for cleaned data
    dv <-  unique(cdata$date)    
    # 5 mins intervals
    iv <-  unique(cdata$interval)    
    # Results data frames    
    # Daily total steps
    dfd <- data.frame()   
    # 5 mins intervals
    dfi <- data.frame()
    # Daily means
    dfm <- data.frame()  
    # Total steps with filled-up NAs
    dff <- data.frame()
```

### What is mean total number of steps taken per day?

#### (1) Make a histogram of the total number of steps taken each day


```r
    # Aggregate sums of steps for each date recursively into results
    for (i in  1:length(dv)){ 
        dfd <- rbind(dfd,(data.frame(date=dv[i],tsteps=sum(cdata[cdata$date==dv[i],]$steps))))
    }
    
    # Histogram showing the frequency of total daily number of steps 
    hist(dfd$tsteps, main=" Total number of steps taken each day", col="yellow", breaks=5, xlab="Number of steps", xlim=c(0,25000),ylim=c(0,30))
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 

#### (2) Calculate and report the mean and median total number of steps taken per day


```r
    cat(paste("Mean of daily steps is: ", round(mean(dfd$tsteps), digits=2)))
```

```
## Mean of daily steps is:  10766.19
```

```r
    cat(paste("Median of daily steps is: ", round(median(dfd$tsteps), digits=2)))
```

```
## Median of daily steps is:  10765
```

### What is the average daily activity pattern?
#### (1) Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
    # Aggregate the mean for every 5 mins interval into the results
    for (i in  1:length(iv)){ 
        dfi <- rbind(dfi,(data.frame(interval=iv[i],msteps=mean(cdata[cdata$interval==iv[i],]$steps))))
    }   
    # Plot the mean value for every 5 min interval across all days
    plot(dfi$interval,dfi$msteps, type="l", main=" Mean value of steps per 5 mins interval", xlab="5 mins intervals", ylab="Average steps", xlim=c(0,2500),ylim=c(0,250), col="blue")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

#### (2) Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
    # 5-minute interval, on average across all the days in the dataset, containing the maximum number of steps
    cat(paste("Interval: ", dfi[dfi$msteps==max(dfi$msteps),]$interval))
```

```
## Interval:  835
```

```r
    cat(paste("Max no. of mean steps per interval: ", round(dfi[dfi$msteps==max(dfi$msteps),]$msteps, digits=2)))
```

```
## Max no. of mean steps per interval:  206.17
```

### Imputing missing values

#### (1) Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with ```NAs```)

```r
    # Calculate and report the total number of missing values in the dataset
    cat(paste("Total number of rows with NAs: ", nrow(rdata) - nrow(cdata)))
```

```
## Total number of rows with NAs:  2304
```

#### (2) Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
*Note: The strategy of filling up ```NAs``` with ```mean()``` for that particular day is not applicable since ```2012-10-01``` is not present in ```complete cases``` cleaned dataset, so we do have to use the other strategy: to replace the ```NAs``` with the ```mean()``` for that particular 5 mins interval*

#### (3) Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
    # Create a new stage data frame to hold mean values for every single day from raw data accordingly with strategy
    sdata <- rdata

    # Scan and fill up the stage data frame with mean values for each 5 min interval (we do already have that info)
    for (i in  1:nrow(sdata)){ 
        if (is.na(sdata[i,]$steps)){
            sdata[i,]$steps=dfi[dfi$interval==(sdata[i,]$interval),]$msteps
        }
    }
```

#### (4) Make a histogram of the total number of steps taken each day


```r
    # Recalculate the total number of steps taken every day based on the new data frame
    
    # Aggregate sums of steps for each date recursively into the results dfd
    for (i in  1:length(rv)){ 
        dff <- rbind(dff,(data.frame(date=rv[i],tsteps=sum(sdata[sdata$date==rv[i],]$steps))))
    }
    # Histogram showing the frequency of total daily number of steps 
    hist(dff$tsteps, main=" Total number of steps taken each day", col="yellow", breaks=5, xlab="Number of steps", xlim=c(0,25000),ylim=c(0,40))
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png) 

#### (5) Report the mean and median total number of steps taken per day


```r
    cat(paste("Mean total steps per day: ", round(mean(dff$tsteps), digits=2)))
```

```
## Mean total steps per day:  10766.19
```

```r
    cat(paste("Median of total steps per day: ",round(median(dff$tsteps), digits=2)))
```

```
## Median of total steps per day:  10766.19
```

#### (6) Do these values differ from the estimates from the first part of the assignment? 

*Yes.*

#### (7) What is the impact of imputing missing data on the estimates of the total daily number of steps?

*The main impact is about influencing the total number of steps (see histograms). As we are introducing mean values on the time intervals as fillers, the qualitative behaviour of the Histogram is not impacted. Noticeable is the fact that the ```median()``` corresponds to the ```mean()```, because of the correspondence of a ```NA``` substituted value positionally as the ```median()``` algorithm demands for an ordered list of values.*

### Are there differences in activity patterns between weekdays and weekends?

#### (1) Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
    # Commodity vector holding unique real used week-days
    wk <- unique(weekdays(as.Date(sdata$date))) 
    
    # Commodity vectors for day classes
    wd <- c(wk[1], wk[2], wk[3], wk[4], wk[5])
    we <- c(wk[6], wk[7])
    
    # New day sets classes
    wz <- c("weekday","weekend")
    
    # Data frame holding day classes (dc) from the filled-up one
    wdata <- sdata
    
    # Append the new column "dc" for day set class
    wdata$dc <- ""
    
    # Scan the filled-in vector and include the new variable day class as "weekday" | "weekend"
    for (i in  1:nrow(sdata)){ 
        if (weekdays(as.Date(sdata[i,]$date)) %in% wd){
            wdata[i,4] <- wz[1]
        }
        else if (weekdays(as.Date(sdata[i,]$date)) %in% we){
            wdata[i,4] <- wz[2]
        }
    }
    
    # Finally convert day class column to factors as requested
    wdata$dc <- as.factor(wdata$dc)
```

```r
cat(paste("New column variable name {dc} is of class: ", class(wdata$dc)))
```

```
## New column variable name {dc} is of class:  factor
```

```r
cat(levels(wdata$dc))
```

```
## weekday weekend
```

#### (2) Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
    # Make plots panel (Plot "looks like" the one provided)
    print(qplot(interval, steps, facets= dc ~ ., data=wdata, geom = "line", xlab="Interval", ylab="Number of Steps", main=c("WeekDays-WeekEnds comparison panel")))
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png) 

