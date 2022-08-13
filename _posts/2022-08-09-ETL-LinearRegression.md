---
layout: post
title: "ETL and Linear Regression using R"
subtitle: "Loading, cleaning, exporting data and applying linear regression to data using R"
background: '/img/posts/etl_lm.jpg'
---
# ETL
Load libraries
```r
library(tidyverse)
library(stringi)
library(readxl)
```

# Load Data

Enter the path of the folder for task files
```r
back_data_dir <- file.path("E:/experiment_data/Backward")
flanker_data_dir <- file.path("E:/experiment_data/Flanker")
stroop_data_dir <- file.path("E:/experiment_data/Stroop")
```

Create a list of all file names
```r
back_file_names <- list.files(back_data_dir,pattern = ".txt", full.names = TRUE)
flanker_file_names <- list.files(flanker_data_dir,pattern = ".txt", full.names = TRUE)
stroop_file_names <- list.files(stroop_data_dir,pattern = ".txt", full.names = TRUE)
```
Read data from all of the files and create a list
```r
data_backward <- lapply(back_file_names, read.table)
data_flanker <- lapply(flanker_file_names, read.table)
data_stroop <- lapply(stroop_file_names, read.table)
```

Change the name of the each data frame of list to the participant name
```r
names(data_backward) <- stri_sub(back_file_names,-9,-5)
names(data_flanker) <- stri_sub(flanker_file_names,-9,-5)
names(data_stroop) <- stri_sub(stroop_file_names,-9,-5)
```



check the list of data frames with their participant name
```r
data_backward
data_flanker
data_stroop
```
Convert list of data frames into one data frame
```r
backward_df <- do.call("rbind", data_backward)
flanker_df <- do.call("rbind", data_flanker)
stroop_df <- do.call("rbind", data_stroop)
```
check single data frame of backward test
```r
str(backward_df)
str(flanker_df)
str(stroop_df)
```
Add participants name along with number of obseravation as column
```r
backward_df <- tibble::rownames_to_column(backward_df,"row_names")
flanker_df <- tibble::rownames_to_column(flanker_df,"row_names")
stroop_df <- tibble::rownames_to_column(stroop_df,"row_names")
```
Change column names of backward task data frame
```r
colnames(backward_df) <- c("participant.No_of_observation","highest_span","Number_of_times","result_backwar_task","Name_of_table_row")
colnames(flanker_df) <- c("participant.No_of_observation","stimulus","congruency","result_flanker_task","response_time")
colnames(stroop_df) <- c("participant.No_of_observation","block_name","name_of_word","color_of_word","match","table_row","key_pressed","answer","response_time_milisecond")
```
# Creating/Saving files

Save backward data frame as csv
```r
write.csv(backward_df,file="backward_data.csv")
```
Save flanker data frame as csv
```r
write.csv(flanker_df,file="flanker_data.csv")
```
Save stroop data frame as csv
```r
write.csv(stroop_df,file="stroop_data.csv")
```
Combine/merge stroop task and flanker task data frames into one data frame
```r
stroop_flanker <- merge(stroop_df,flanker_df, by = "participant.No_of_observation" , all= TRUE)
```
Now, combine/merge the backward data frame to previously merged stroop and flanker data
```r
experiment_data <- merge(stroop_flanker,backward_df,by ="participant.No_of_observation", all = TRUE)
```
Save the combined data of backward, flanker and stroop data as csv file
```r
write_csv(experiment_data, file="experiment_data.csv")
```
## Calculate score of each task corresponding to relevant participant


Initialize an empty data frame for task score data of participants
```r
df_back_score <- data.frame(matrix(ncol = 2, nrow = 0))
df_flank_score <- data.frame(matrix(ncol = 2, nrow = 0))
df_stroop_score <- data.frame(matrix(ncol = 2, nrow = 0))
```
### loop through each list of task data to calculate the percentage score of participants in each task

Loop 1 : For backward task
```r
for (i in 1:length(data_backward)){
  
  #### get result variable from each data frame of participants
  x <- data_backward[[i]]$V3
  
  #### count number of occurrences for each type of result
  x_cout <- table(x)
  
  #### calculate percentage for each type(correct or wrong) of result 
  x_percent <- x_cout/length(x)
  
  #### convert the table containing the percentage of result to data frame
  x_score <- data.frame(x_percent)
  
  #### get the name of the participant
  name <- names(data_backward[i])
  
  #### add participant name to its score for reference
  x_score$participant <- name
  
  #### create a single data frame for all of the participants
  df_back_score <- rbind(df_back_score, x_score)
  
}
```
Loop 2: For flanker task
```r
for (i in 1:length(data_flanker)){
  
  #### get result variable from each data frame of participants
  x <- data_flanker[[i]]$V3
  
  #### count number of occurrences for each type of result
  x_cout <- table(x)
  
  #### calculate percentage for each type(correct or wrong) of result 
  x_percent <- x_cout/length(x)
  
  #### convert the table containing the percentage of result to data frame
  x_score <- data.frame(x_percent)
  
  #### get the name of the participant
  name <- names(data_flanker[i])
  
  #### add participant name to its score for reference
  x_score$participant <- name
  
  #### create a single data frame for all of the participants
  df_flank_score <- rbind(df_flank_score, x_score)
  
}
```
Loop 3: For stroop task
```r
for (i in 1:length(data_stroop)){
  
  #### get result variable from each data frame of participants
  x <- data_stroop[[i]]$V7
  
  #### count number of occurrences for each type of result
  x_cout <- table(x)
  
  #### calculate percentage for each type(correct or wrong) of result 
  x_percent <- x_cout/length(x)
  
  #### convert the table containing the percentage of result to data frame
  x_score <- data.frame(x_percent)
  
  #### get the name of the participant
  name <- names(data_stroop[i])
  
  #### add participant name to its score for reference
  x_score$participant <- name
  
  #### create a single data frame for all of the participants
  df_stroop_score <- rbind(df_stroop_score, x_score)
  
}
```
### change variable names for better understanding
```r
colnames(df_back_score) <- c("result", "back_score_percentage","participant")
colnames(df_flank_score) <- c("result", "flank_score_percentage","participant")
colnames(df_stroop_score) <- c("result", "stroop_score_percentage","participant")
```
### getting the data of participant where result is correct
```r
df_correct <- df_back_score[df_back_score$result==1,]
df_flank_score_correct <- df_flank_score[df_flank_score$result==1,]
df_stroop_correct <- df_stroop_score[df_stroop_score$result==1,]
```
### rounding the percentage of the result to two digits
```r
df_correct2 <- df_correct %>%
  mutate_if(is.numeric,
            round,
            digits = 2)

df_stroop_correct2 <- df_stroop_correct %>%
  mutate_if(is.numeric,
            round,
            digits = 2)
```
## Combine scores of participants with their age and IQ data

read excel file of age and IQ data of participants
```r
general_data <- read_excel("Age, IQ, Language.xlsx")
```
change column names 
```r
colnames(general_data) <- c("participant", "age", "language", "IQ")
```
merge participants general data with backward, flanker and stroop task score
```r
data_b <- merge(general_data, df_correct2, by ="participant", all = TRUE )
data_f <- merge(data_b, df_flank_score_correct, by = "participant" , all = TRUE)
data <- merge(data_f, df_stroop_correct2, by = "participant", all = TRUE)
```
remove unnecessary variables
```r
data_new <- subset(data, select = -c(5,7,9))
```
Some rows have NA value, replacing it with 0
```r
data_new$back_score_percentage[is.na(data_new$back_score_percentage)] <- 0
data_new$flank_score_percentage[is.na(data_new$flank_score_percentage)] <- 0
data_new$stroop_score_percentage[is.na(data_new$stroop_score_percentage)] <- 0
```
# Linear regression models

Linear regression model for backward task
```r
backward_linear_model<- lm(back_score_percentage ~ age+IQ, data = data_new )
```
Linear regression model for flanker task
```r
flanker_linear_model <- lm(flank_score_percentage ~ age + IQ, data = data_new )
```
Linear regression model for stroop task
```r
stroop_linear_model <- lm(stroop_score_percentage ~ age + IQ, data = data_new)
```







