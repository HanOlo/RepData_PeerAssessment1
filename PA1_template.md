---
title: "Reproducible Research: Peer Assessment 1"
author: "Hans"
date: "9/25/2021"
output: 
  html_document:
    keep_md: true
---



## Libraries


```r
library(tidyverse)
```

```
## -- Attaching packages --------------------------------------- tidyverse 1.3.1 --
```

```
## v ggplot2 3.3.5     v purrr   0.3.4
## v tibble  3.1.4     v dplyr   1.0.7
## v tidyr   1.1.3     v stringr 1.4.0
## v readr   2.0.1     v forcats 0.5.1
```

```
## -- Conflicts ------------------------------------------ tidyverse_conflicts() --
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(lubridate)
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following objects are masked from 'package:base':
## 
##     date, intersect, setdiff, union
```



## Load data


```r
activity <- read_csv("activity.csv")
```

```
## Rows: 17568 Columns: 3
```

```
## -- Column specification --------------------------------------------------------
## Delimiter: ","
## dbl  (2): steps, interval
## date (1): date
```

```
## 
## i Use `spec()` to retrieve the full column specification for this data.
## i Specify the column types or set `show_col_types = FALSE` to quiet this message.
```


## Tasks to be completed

1. Code for reading in the dataset and/or processing the data

2. Histogram of the total number of steps taken each day

3. Mean and median number of steps taken each day

4. Time series plot of the average number of steps taken

5. The 5-minute interval that, on average, contains the maximum number of steps

6. Code to describe and show a strategy for imputing missing data

7. Histogram of the total number of steps taken each day after missing values are imputed

8. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends

9. All of the R code needed to reproduce the results (numbers, plots, etc.) in the report




### 2. Histogram of the total number of steps taken each day

```r
prob2 <- activity %>% 
group_by(date) %>% 
summarize(total = sum(steps)) 

#png(filename="plot1.png")
prob2 %>% 
ggplot() +
geom_col(aes(x=date, y=total ))+
  theme_classic()
```

```
## Warning: Removed 8 rows containing missing values (position_stack).
```

![Image1](plot1.png)  <!-- -->

```r
#dev.off()
```




### 3. Mean and median number of steps taken each day

The question asks for the mean and median for **each day**.  Grouped by date and plotted on a chart for ease of viewing.  There are a lot of 0 data points throughout the day which causes the median points to be along the bottom.  

```r
#png(filename="plot2.png")
activity %>% 
group_by(date) %>% 
summarize(mean = mean(steps),
          median = median(steps))%>% 
  ggplot() +
  geom_point(aes(date, mean, color = "Mean") )+
  geom_point(aes(date, median, color = "Median") ) +
  theme_bw() +
  labs(title = "Mean and Median Steps per Day",
       y = "Steps",
       x = "Date")
```

```
## Warning: Removed 8 rows containing missing values (geom_point).

## Warning: Removed 8 rows containing missing values (geom_point).
```

![Image3](plot2.png)  <!-- -->

```r
#dev.off()
```




### 4. Time series plot of the average number of steps taken


```r
#png(filename = "plot3.png")
activity %>% 
group_by(date) %>% 
summarize(avg = mean(steps)) %>% 
ggplot() +
geom_point(aes(x=date, y=avg), color = "blue")+
  theme_bw() +
  labs(title = "Average Steps Taken by Date",
       y = "Average Steps",
       x= "Date")
```

```
## Warning: Removed 8 rows containing missing values (geom_point).
```

![Image4](plot3.png)  <!-- -->

```r
#dev.off()
```



### 5. The 5-minute interval that, on average, contains the maximum number of steps
#### The maximum 5 minute average interval was 2355.

```r
activity %>% 
  group_by(interval) %>% 
  summarize(mean = mean(steps, na.rm = TRUE)) %>% 
  max()
```

```
## [1] 2355
```



### 6. Code to describe and show a strategy for imputing missing data

The data set contains 2 months of data broken down into 5 minute increments. 
Step 1: check dates to determine there is a pattern to the missing data
Findings: 8 days missing 288 records each
Step 2: group by day of the week and determine average number of steps per 5 minute increment.
Step 3: Impute data for missing day of the week



```r
activity %>% 
  filter(is.na(steps)) %>% 
  group_by(date) %>% 
  summarise(miss = n())
```

```
## # A tibble: 8 x 2
##   date        miss
##   <date>     <int>
## 1 2012-10-01   288
## 2 2012-10-08   288
## 3 2012-11-01   288
## 4 2012-11-04   288
## 5 2012-11-09   288
## 6 2012-11-10   288
## 7 2012-11-14   288
## 8 2012-11-30   288
```

```r
prob6 <- activity %>% 
  mutate(day = wday(date, label = TRUE)) %>% 
  group_by(day, interval) %>% 
  summarize(avg = mean(steps, na.rm=TRUE))
```

```
## `summarise()` has grouped output by 'day'. You can override using the `.groups` argument.
```

```r
prob6_2 <- activity %>% 
  mutate(day = wday(date, label = TRUE))

temp <- prob6_2 %>% 
  filter(date %in% c(ymd('2012-10-01', '2012-10-08', '2012-11-01', '2012-11-04', '2012-11-09', '2012-11-10', '2012-11-14', '2012-11-30'))) %>% 
  right_join(prob6) %>% 
  mutate(steps = avg) %>% 
  select(!avg)
```

```
## Joining, by = c("interval", "day")
```

```r
prob_6_3 <- prob6_2 %>% 
  filter(!date %in% c(ymd('2012-10-01', '2012-10-08', 
                          '2012-11-01', '2012-11-04', 
                          '2012-11-09', '2012-11-10', 
                          '2012-11-14', '2012-11-30'))) %>% 
  bind_rows(temp)
```



### 7. Histogram of the total number of steps taken each day after missing values are imputed


```r
#png(filename = "plot4.png")
prob_6_3 %>% 
group_by(date) %>% 
summarize(total = sum(steps)) %>% 
ggplot() +
geom_col(aes(x=date, y=total ))+
  theme_classic()
```

```
## Warning: Removed 1 rows containing missing values (position_stack).
```

![Image5](plot4.png)<!-- -->

```r
#dev.off()
```




### 8. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends



```r
wkday <- c("Mon", "Tue", "Wed", "Thu", "Fri")
wkend <- c("Sat", "Sun")

prob_7_a <- prob_6_3 %>% 
  filter(day %in% wkday) %>% 
  mutate(wk = "Weekday")

prob_7_b <- prob_6_3 %>% 
  filter(day %in% wkend) %>% 
  mutate(wk = "Weekend")

prob_7 <- bind_rows(prob_7_a, prob_7_b) %>% 
  group_by(interval, wk) %>% 
  summarize(avg = mean(steps))
```

```
## `summarise()` has grouped output by 'interval'. You can override using the `.groups` argument.
```

```r
#png(filename = "plot5.png")
  prob_7 %>% 
  ggplot() +
  geom_point(aes(x=interval, y=avg, color = wk) )+
  theme_bw() +
  labs(title = "Average Steps Taken by Interval",
       y = "Average Steps",
       x= "Interval") +
  facet_wrap(~wk, dir="v")
```

![Image5](plot5.png)<!-- -->

```r
#  dev.off()
```



