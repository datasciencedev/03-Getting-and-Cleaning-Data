run_analysis
============
Updated at 2015-02-22 19:18:37.


Instructions for project
------------------------

> The purpose of this project is to demonstrate your ability to collect, work with, and clean a data set. The goal is to prepare tidy data that can be used for later analysis. You will be graded by your peers on a series of yes/no questions related to the project. You will be required to submit: 1) a tidy data set as described below, 2) a link to a Github repository with your script for performing the analysis, and 3) a code book that describes the variables, the data, and any transformations or work that you performed to clean up the data called CodeBook.md. You should also include a README.md in the repo with your scripts. This repo explains how all of the scripts work and how they are connected.  
> 
> One of the most exciting areas in all of data science right now is wearable computing - see for example this article . Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone. A full description is available at the site where the data was obtained: 
> 
> http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones 
> 
> Here are the data for the project: 
> 
> https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip 
> 
> You should create one R script called run_analysis.R that does the following. 
> 
> 1. Merges the training and the test sets to create one data set.
> 2. Extracts only the measurements on the mean and standard deviation for each measurement.
> 3. Uses descriptive activity names to name the activities in the data set.
> 4. Appropriately labels the data set with descriptive activity names.
> 5. Creates a second, independent tidy data set with the average of each variable for each activity and each subject. 


Project Setup
-------------
Set working directory.

```r
wd <- getwd()
setwd("/Users/bobfridley/Documents/git/Getting-and-Cleaning-Data/Course-Project")
```

Load packages using sapply.


```r
packages <- c("data.table", "reshape2", "dplyr")
loadp <- sapply(packages, library, character.only=TRUE, quietly=TRUE,
        warn.conflicts=FALSE, logical.return=TRUE, verbose=FALSE)

## stop script if any packages are not loaded

if (!any(loadp)) {
        stop("unable to load required packages")
}
```

Get the data
------------
Set filepath for downloaded file.


```r
dataDirectory <- "./UCI HAR Dataset"
destFile <- "getdata-projectfiles-UCI HAR Dataset.zip"
destFilePath <- paste(dataDirectory, destFile, sep = "/")

## create data directory if not exists

if (!file.exists(dataDirectory)) {
        dir.create(dataDirectory)
}
```

Download and unzip the file to the `UCI HAR Dataset` directory.


```r
if (!file.exists(destFilePath)) {
        fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
        download.file(fileUrl, destfile=destFilePath, method="curl", mode="wb")
        unzip (destFilePath)
}
```

List the files in `UCI HAR Dataset` directory.
----------------------------------------------


```r
list.files(dataDirectory, recursive=TRUE)
```

```
##  [1] "activity_labels.txt"                         
##  [2] "features_info.txt"                           
##  [3] "features.txt"                                
##  [4] "getdata-projectfiles-UCI HAR Dataset.zip"    
##  [5] "README.txt"                                  
##  [6] "test/Inertial Signals/body_acc_x_test.txt"   
##  [7] "test/Inertial Signals/body_acc_y_test.txt"   
##  [8] "test/Inertial Signals/body_acc_z_test.txt"   
##  [9] "test/Inertial Signals/body_gyro_x_test.txt"  
## [10] "test/Inertial Signals/body_gyro_y_test.txt"  
## [11] "test/Inertial Signals/body_gyro_z_test.txt"  
## [12] "test/Inertial Signals/total_acc_x_test.txt"  
## [13] "test/Inertial Signals/total_acc_y_test.txt"  
## [14] "test/Inertial Signals/total_acc_z_test.txt"  
## [15] "test/subject_test.txt"                       
## [16] "test/X_test.txt"                             
## [17] "test/y_test.txt"                             
## [18] "tidydata.txt"                                
## [19] "train/Inertial Signals/body_acc_x_train.txt" 
## [20] "train/Inertial Signals/body_acc_y_train.txt" 
## [21] "train/Inertial Signals/body_acc_z_train.txt" 
## [22] "train/Inertial Signals/body_gyro_x_train.txt"
## [23] "train/Inertial Signals/body_gyro_y_train.txt"
## [24] "train/Inertial Signals/body_gyro_z_train.txt"
## [25] "train/Inertial Signals/total_acc_x_train.txt"
## [26] "train/Inertial Signals/total_acc_y_train.txt"
## [27] "train/Inertial Signals/total_acc_z_train.txt"
## [28] "train/subject_train.txt"                     
## [29] "train/X_train.txt"                           
## [30] "train/y_train.txt"
```

Read the files.
---------------

Read Subject files into data tables.


```r
dtSubjectTrain <- fread(file.path(dataDirectory, "train", "subject_train.txt"))
dtSubjectTest <- fread(file.path(dataDirectory, "test", "subject_test.txt"))
```

Read Activity files into data tables.


```r
dtActivityTrain <- fread(file.path(dataDirectory, "train", "Y_train.txt"))
dtActivityTest <- fread(file.path(dataDirectory, "test", "Y_test.txt"))
```

Read the data files.
Used read.table/data.table since fread was unsuccessful.
1. Read data file into data.frame.
2. Convert data.frame into data.table.


```r
temp <- read.table(file.path(dataDirectory, "train", "X_train.txt"),
        header=FALSE)
dtTrain <- data.table(temp)
temp <- read.table(file.path(dataDirectory, "test", "X_test.txt"),
        header=FALSE)
dtTest <- data.table(temp)
```

Merge the training and the test datasets.
-----------------------------------------

Concatenate the data tables.


```r
dtSubject <- rbind(dtSubjectTrain, dtSubjectTest)
setnames(dtSubject, "V1", "subject")
dtActivity <- rbind(dtActivityTrain, dtActivityTest)
setnames(dtActivity, "V1", "activitynum")
dt <- rbind(dtTrain, dtTest)
```

Merge columns.


```r
dtTrainTest <- cbind(dtSubject, dtActivity)
dt <- cbind(dtTrainTest, dt)
```

Set key.


```r
setkey(dt, subject, activitynum)
```


Extract only the mean and standard deviation
--------------------------------------------

Read the `features.txt` file. This tells which variables in `dt` are measurements for the mean and standard deviation.


```r
dtFeatures <- fread(file.path(dataDirectory, "features.txt"))
setnames(dtFeatures, names(dtFeatures), c("featurenum", "featurename"))
```

Subset only measurements for the mean and standard deviation.


```r
dtFeatures <- dtFeatures[grepl("mean\\(\\)|std\\(\\)", featurename)]
```

Convert the column numbers to a vector of variable names matching columns in `dt`.


```r
dtFeatures$featurecode <- dtFeatures[, paste0("V", featurenum)]
head(dtFeatures)
```

```
##    featurenum       featurename featurecode
## 1:          1 tBodyAcc-mean()-X          V1
## 2:          2 tBodyAcc-mean()-Y          V2
## 3:          3 tBodyAcc-mean()-Z          V3
## 4:          4  tBodyAcc-std()-X          V4
## 5:          5  tBodyAcc-std()-Y          V5
## 6:          6  tBodyAcc-std()-Z          V6
```

```r
dtFeatures$featurecode
```

```
##  [1] "V1"   "V2"   "V3"   "V4"   "V5"   "V6"   "V41"  "V42"  "V43"  "V44" 
## [11] "V45"  "V46"  "V81"  "V82"  "V83"  "V84"  "V85"  "V86"  "V121" "V122"
## [21] "V123" "V124" "V125" "V126" "V161" "V162" "V163" "V164" "V165" "V166"
## [31] "V201" "V202" "V214" "V215" "V227" "V228" "V240" "V241" "V253" "V254"
## [41] "V266" "V267" "V268" "V269" "V270" "V271" "V345" "V346" "V347" "V348"
## [51] "V349" "V350" "V424" "V425" "V426" "V427" "V428" "V429" "V503" "V504"
## [61] "V516" "V517" "V529" "V530" "V542" "V543"
```

Subset these variables using variable names.


```r
select <- c(key(dt), dtFeatures$featurecode)
dt <- dt[, select, with=FALSE]
```


Use descriptive activity names.
-------------------------------

Read `activity_labels.txt` file. This will be used to add descriptive names to the activities.


```r
dtActivities <- fread(file.path(dataDirectory, "activity_labels.txt"))
setnames(dtActivities, names(dtActivities), c("activitynum", "activityname"))
```


Label with descriptive activity names.
--------------------------------------

Merge activity labels.


```r
dt <- merge(dt, dtActivities, by="activitynum", all.x=TRUE)
```

Add `activityname` as a key.


```r
setkey(dt, subject, activitynum, activityname)
```

Melt the data table to reshape it to a tall and narrow format.


```r
dt <- data.table(melt(dt, key(dt), variable.name="featurecode"))
```

Merge activity name.


```r
dt <- merge(dt, dtFeatures[, list(featurenum, featurecode, featurename)],
        by="featurecode", all.x=TRUE)
```

Create a new variable, `activity` that is equivalent to `activityname` as a factor class.
Create a new variable, `feature` that is equivalent to `featurename` as a factor class.


```r
dt$activity <- factor(dt$activityname)
dt$feature <- factor(dt$featurename)
```

Create new variables from `featurename`


```r
## Features having 1 category
dt$jerk <- factor(grepl("Jerk", dt$feature), labels=c(NA, "Jerk"))
dt$magnitude <- factor(grepl("Mag", dt$feature), labels=c(NA, "Magnitude"))

## Features having 2 categories
n <- 2
y <- matrix(seq(1, n), nrow=n)
x <- matrix(c(grepl("^t", dt$feature), grepl("^f", dt$feature)), ncol=nrow(y))
dt$domain <- factor(x %*% y, labels=c("Time", "Freq"))
x <- matrix(c(grepl("Acc", dt$feature), grepl("Gyro", dt$feature)), ncol=nrow(y))
dt$instrument <- factor(x %*% y, labels=c("Accelerometer", "Gyroscope"))
x <- matrix(c(grepl("BodyAcc", dt$feature), grepl("GravityAcc", dt$feature)), ncol=nrow(y))
dt$acceleration <- factor(x %*% y, labels=c(NA, "Body", "Gravity"))
x <- matrix(c(grepl("mean()", dt$feature), grepl("std()", dt$feature)), ncol=nrow(y))
dt$variable <- factor(x %*% y, labels=c("Mean", "SD"))

## Features having 3 categories
n <- 3
y <- matrix(seq(1, n), nrow=n)
x <- matrix(c(grepl("-X", dt$feature), grepl("-Y", dt$feature), grepl("-Z", dt$feature)), ncol=nrow(y))
dt$axis <- factor(x %*% y, labels=c(NA, "X", "Y", "Z"))
```

Create a tidy data set.
-----------------------

Create a data set with the average of each variable for each activity and each subject.


```r
setkey(dt, subject, activity, domain, acceleration, instrument,
        jerk, magnitude, variable, axis)
dtTidy <- dt[, list(count = .N, average = mean(value)), by=key(dt)]
```

Make codebook.


```r
knit("make_codebook.Rmd", output="codebook.md", encoding="UTF-8", quiet=TRUE)
```

```
## [1] "codebook.md"
```

```r
markdownToHTML("codebook.md", "codebook.html")
```