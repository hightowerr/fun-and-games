---
layout: posts
title:  "Getting and cleaning data using R"
date:   2021-11-01 13:00:00 +0100
categories: cleaning
author: Olayinka Ola
---
The purpose of this project is to demonstrate my ability to collect, work with, and clean a data set.

## **Getting and Cleaning Data Course Project**

The purpose of this project was to demonstrate the ability to collect, work with, and clean a data set. The goal being to prepare tidy data that can be used for later analysis. I submitted the following:

1. a tidy data set as described below
2. a link to a Github repository with my script for performing the analysis, and a code book that describes the variables, the data, and any transformations or work that I performed to clean up the data called CodeBook.md. I also include a README.md in the repo with my scripts. This repo explains how all of the scripts work and how they are connected.


[Here][Here] is where the dataset, description and how the data was obtained.
Link to my [Github Repo][Github Repo]

## **Libraries used**

- `dplyr` is used to aggregate variables to create the tidy data.
- `plyr` to split data apart, do stuff to it, and mash it back together
- `stringr` make working with strings as easy as possible
- `knitr` to create reproducible web-based reports.

```r
library(stringr)
library(dplyr)
library(plyr)
library(knitr)
```

## 1. Download the dataset
```r
if (!(file.exists("data")))
{ dir.create("data")}
if (!(file.exists("data/Dataset.zip")))
{ download.file("https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip",destfile ="data/Dataset.zip", method="curl")}
if (!(file.exists("data/UCI HAR Dataset")))
{unzip("data/Dataset.zip",exdir="data")}
```

## 2. Merging the training and the test sets to create one data set.
### Read the Subject files

```r
subject_train <- read.table("data/UCI HAR Dataset/train/subject_train.txt", header = FALSE, sep = "")
subject_test <- read.table("data/UCI HAR Dataset/test/subject_test.txt", header = FALSE, sep = "")
sub_bind <- rbind(subject_train, subject_test)
```

### Read the Activity files
```r
y_train <- read.table("data/UCI HAR Dataset/train/y_train.txt", header = FALSE, sep = "")
y_test <- read.table("data/UCI HAR Dataset/test/y_test.txt", header = FALSE, sep = "")
y_bind <- rbind(y_train, y_test)
```

### Read Features files
```r
X_train <- read.table("data/UCI HAR Dataset/train/X_train.txt", header = FALSE, sep = "")
X_test <- read.table("data/UCI HAR Dataset/test/X_test.txt", header = FALSE, sep = "")
X_bind <- rbind(X_train, X_test)

features <- read.table("data/UCI HAR Dataset/features.txt",header = FALSE, sep = "")

names(sub_bind) <- c("subject")
names(y_bind) <- c("activity")
names(X_bind) <- features$V2
dataset <- cbind(sub_bind, X_bind, y_bind)
```

## 3. Extracts only the measurements on the mean and standard deviation for each measurement.
```r
tidy_data <- dataset[, grep(pattern = "-mean\\(\\)|-std\\(\\)", colnames(dataset))]
new_data <- cbind(sub_bind, y_bind, tidy_data)
```

## 4. Useing descriptive activity names to name the activities in the data set
```r
new_data$activity[new_data$activity == "1"] <- "WALKING"
new_data$activity[new_data$activity == "2"] <- "WALKING_UPSTAIRS"
new_data$activity[new_data$activity == "3"] <- "WALKING_DOWNSTAIRS"
new_data$activity[new_data$activity == "4"] <- "SITTING"
new_data$activity[new_data$activity == "5"] <- "STANDING"
new_data$activity[new_data$activity == "6"] <- "LAYING"
```

## 5. Appropriately labels the data set with descriptive variable names.
```r
names(new_data) <- str_replace_all(names(new_data), c("^t"="time", "^f"="frequency", "Acc"="Accelerometer", "Gyro"="Gyroscope", "Mag"="Magnitude", "BodyBody"="Body"))
```

### 6. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.
```r
data_group <- new_data %>% group_by(subject, activity) %>% dplyr::summarise(across(everything(), list(mean)))
data_group <- data_group[order(data_group$subject, data_group$activity),]
write.table(data_group, file = "tidydata.txt", row.names = FALSE)
```

### 7. Produce a codebook
```r
knit2html("./run_analysis.Rmd");
```

[Here]: http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones
[Github Repo]: https://github.com/hightowerr/datasciencecoursera/tree/master/Getting%20and%20cleaning%20Data
