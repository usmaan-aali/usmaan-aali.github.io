---
layout: post
title: "Data cleaning in R"
subtitle: "Loading, cleaning and exporting data in R"
background: '/img/posts/data cleaning.jpeg'
---
 
#### First of all install required packages
**tidyverse for data import and wrangling\
lubridate for date functions\
ggplot for visualization**

```r
install.packages("tidyverse")
install.packages("lubridate")
install.packages("ggplot2")

library(tidyverse)      #helps wrangle data
library(lubridate)      #helps wrangle date attributes
library(ggplot2)        #helps visualize data
getwd()                 #displays your working directory
setwd("C:/Users/shan9/Desktop/Cyclistics/Cyclistic_Data_CSV") #sets your working directory to simplify calls to data
```
### STEP 1: COLLECT DATA

**Upload Cyclistics Datasets csv files here**
```r
apr21 <- read_csv("202104-divvy-tripdata.csv")
may21 <- read_csv("202105-divvy-tripdata.csv")
jun21 <- read_csv("202106-divvy-tripdata.csv")
jul21 <- read_csv("202107-divvy-tripdata.csv")
aug21 <- read_csv("202108-divvy-tripdata.csv")
sep21 <- read_csv("202109-divvy-tripdata.csv")
oct21 <- read_csv("202110-divvy-tripdata.csv")
nov21 <- read_csv("202111-divvy-tripdata.csv")
dec21 <- read_csv("202112-divvy-tripdata.csv")
jan22 <- read_csv("202201-divvy-tripdata.csv")
feb22 <- read_csv("202202-divvy-tripdata.csv")
mar22 <- read_csv("202203-divvy-tripdata.csv")
```
### STEP 2: WRANGLE DATA AND COMBINE INTO A SINGLE FILE
```r
str(apr21)
str(may21)
str(jun21)
str(jul21)
str(aug21)
str(sep21)
str(oct21)
str(nov21)
str(dec21)
str(jan22)
str(feb22)
str(mar22)
```
**Change the datatype of date columns**
```r
dec21 <- dec21 %>% 
  mutate(started_at = mdy_hms(started_at), ended_at = mdy_hms(ended_at))
```

**Stack individual quarter data frames into one big data frame**
```r
data_apr21_mar22 <- bind_rows(apr21, may21, jun21,jul21,aug21,sep21,oct21,nov21,dec21,jan22,feb22,mar22)
```


### STEP 3: CLEAN UP AND ADD DATA TO PREPARE FOR ANALYSIS
**Inspect the new table that has been created**
```r
colnames(data_apr21_mar22)  #List of column names
nrow(data_apr21_mar22)  #How many rows are in data frame?
dim(data_apr21_mar22) #dimensions of data frame
summary(data_apr21_mar22) #Statistical summary of data
```

**Add a "ride_length" calculation to data_apr21_mar22 (in seconds)**
```r
data_apr21_mar22$ride_length <- difftime(data_apr21_mar22$ended_at, data_apr21_mar22$started_at)
str(data_apr21_mar22)
```
**Check "ride_length" to confirm it's numeric datatype so we can run calculations on the data**
```r
is.double(data_apr21_mar22$ride_length)
```
**Checking the data for negative ride_length values**
```r
table(data_apr21_mar22$ride_length < 0)
```
#### Remove "bad" data
**The dataframe includes a few hundred entries when bikes were taken out of docks and checked for quality by Divvy or ride_length was negative**
```r
data_apr21_mar22_V2 <- data_apr21_mar22[!(data_apr21_mar22$ride_length < 0),]
```
**Confirming the removal of bad data** 
```r
table(data_apr21_mar22_V2$ride_length < 0)
```
**Add columns that list the day of week of each ride**\
This will allow us to aggregate ride data for each day of week, before completing these operations we could only aggregate at the ride level
```r
data_apr21_mar22_V2$day_of_week <- format(as.Date(data_apr21_mar22_V2$started_at), "%A")

data_apr21_mar22_V2$time_of_day <- format(data_apr21_mar22_V2$started_at, format = "%H")
```
### STEP 4: CONDUCT DESCRIPTIVE ANALYSIS

**Descriptive analysis on ride_length (all figures in seconds)**
```r
mean(data_apr21_mar22_V2$ride_length) # average
median(data_apr21_mar22_V2$ride_length) # midpoint number
min(data_apr21_mar22_V2$ride_length) # shortest ride
max(data_apr21_mar22_V2$ride_length) # longest ride

summary(data_apr21_mar22_V2$ride_length)
```
**Compare members and casual users**
```r
aggregate(data_apr21_mar22_V2$ride_length ~ data_apr21_mar22_V2$member_casual, FUN = mean)
aggregate(data_apr21_mar22_V2$ride_length ~ data_apr21_mar22_V2$member_casual, FUN = median)
aggregate(data_apr21_mar22_V2$ride_length ~ data_apr21_mar22_V2$member_casual, FUN = max)
aggregate(data_apr21_mar22_V2$ride_length ~ data_apr21_mar22_V2$member_casual, FUN = min)
```
**See the average ride time by each day for members vs casual users**
```r
aggregate(data_apr21_mar22_V2$ride_length ~ data_apr21_mar22_V2$member_casual + data_apr21_mar22_V2$day_of_week, FUN = mean)
```
**Notice that the days of the week are out of order. Let's fix that.**
```r
data_apr21_mar22_V2$day_of_week <- ordered(data_apr21_mar22_V2$day_of_week, levels=c("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"))
```
**Now, let's run the average ride time by each day for members vs casual users**
```r
aggregate(data_apr21_mar22_V2$ride_length ~ data_apr21_mar22_V2$member_casual + data_apr21_mar22_V2$day_of_week, FUN = mean)
```

**Analyze ridership data by type and weekday**
```r
data_by_type_wday <- data_apr21_mar22_V2 %>%
  mutate(weekday = wday(started_at, label = TRUE)) %>%  #creates weekday field using wday()
  group_by(member_casual, weekday) %>%  #groups by usertype and weekday
  summarise(number_of_rides = n()							#calculates the number of rides and average duration 
            ,average_duration = mean(ride_length)) %>% 		# calculates the average duration
  arrange(member_casual, weekday)	
```

**Let's visualize the number of rides by rider type**
```r
data_by_type_wday %>% 
  ggplot(aes(x = weekday, y = number_of_rides, fill = member_casual)) +
  geom_col(position = "dodge")
```
**Let's visualize the average ride duration by rider type**
```r
data_by_type_wday %>%
  ggplot(aes(x = weekday, y = average_duration, fill = member_casual))+
  geom_col(position = "dodge")
```
### STEP 5: EXPORT SUMMARY FILE FOR FURTHER ANALYSIS

**Create a csv file that we will visualize in Excel, Tableau, or any presentation software**
```r
counts <- aggregate(data_apr21_mar22_V2$ride_length ~ data_apr21_mar22_V2$member_casual + data_apr21_mar22_V2$day_of_week, FUN = mean)
write.csv(counts, file = "avg_ride_length.csv")

num_rides <- aggregate(data_apr21_mar22_V2$ride_id ~ data_apr21_mar22_V2$member_casual + data_apr21_mar22_V2$day_of_week, FUN = length)
write_csv(num_rides,file = "num_rides_by_day_of_week.csv")

rides_by_time <- aggregate(data_apr21_mar22_V2$ride_id ~ data_apr21_mar22_V2$member_casual + data_apr21_mar22_V2$started_at, FUN = length)
write_csv(rides_by_time, file = "num_rides_by_time_of_day.csv")

type_bike <- aggregate(data_apr21_mar22_V2$ride_id ~ data_apr21_mar22_V2$member_casual + data_apr21_mar22_V2$rideable_type, FUN = length)
write_csv(type_bike, file = "type_of_bike_used_user_type.csv")
```