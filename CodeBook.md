---
title: "CodeBook.md"
author: "Pat Lapomarda"
date: "August 20, 2015"
output: html_document
---

The first part of the [program](run_analysis.R) sets the working directory and downloads the raw data:

```{r}
##Dowload Data
setwd("~/Documents/Class/Getting and Cleaning Data")
if(!file.exists("./data")){dir.create("./data")}
fileURL <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileURL,destfile="./data/dataset.zip",method="curl")
```

The raw data is compressed, so it must be unzipped:

```{r}
##Unzip Downloaded Data
unzip("./data/dataset.zip",exdir="./data")
```

Once uncompressed, the raw data can be loaded to R:

```{r}
##Load Test & Train Data
Train <- read.table("./data/UCI HAR Dataset/train/X_train.txt")
Test <- read.table("./data/UCI HAR Dataset/test/X_test.txt")


##Load Test & Train Activity/Feature/Subject IDs & Labels
TrainID <- read.table("./data/UCI HAR Dataset/train/y_train.txt")
TestID <- read.table("./data/UCI HAR Dataset/test/y_test.txt")
Activity <- read.table("./data/UCI HAR Dataset/activity_labels.txt")
names(Activity) <- c("ActivityID","Activity")
Feature <- read.table("./data/UCI HAR Dataset/features.txt")
SubjectTrain <- read.table("./data/UCI HAR Dataset/train/subject_train.txt")
SubjectTest <- read.table("./data/UCI HAR Dataset/test/subject_test.txt")
```

The raw data is then combined to form a consolidated data frame that can be analyzed:

```{r}
#Join Test & Train Data & IDs
AllData <- rbind(Test,Train)
AllID <- rbind(TestID,TrainID)
SubjectData <- rbind(SubjectTest,SubjectTrain)

#Add Labels to Data
names(AllData) <- as.vector(Feature[,2])
names(AllID) <- c("ActivityID")
names(SubjectData) <- c("SubjectID")

#Consolidate IDs and Data
FitData <- cbind(AllID,SubjectData,AllData)

#Replace ActivityID with Activity and rename
FitData$ActivityID <- mapvalues(FitData$ActivityID,as.vector(Activity$ActivityID),as.vector(Activity$Activity))
names(FitData)[1] <- "Activity"
```

Finally, the consolidated data frame is reshaped to ease the calculation of summary statistics and written out:
```{r}
#Create Summary Data: Mean by Activity, Subject, & Measure
install.packages("reshape")
library(reshape)
MoltenData <- melt(FitData,id.vars=c("Activity","SubjectID"))
names(MoltenData)[3] <- "Measure"
SummaryData <- ddply(MoltenData,.(Activity,SubjectID,Measure),summarise,Mean=mean(value))
write.table(SummaryData,file="./data/SummaryData.txt",row.names=FALSE)
```

The final output contains the average of each variable for each activity and each subject.  The detailed descriptions of the variables, activities, and raw data files can be found in the [UCI HAR Dataset.names](http://archive.ics.uci.edu/ml/machine-learning-databases/00240/UCI%20HAR%20Dataset.names) file contained within the [raw data archive](http://archive.ics.uci.edu/ml/machine-learning-databases/00240/UCI%20HAR%20Dataset.zip).