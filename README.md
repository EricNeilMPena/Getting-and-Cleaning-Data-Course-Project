# Getting-and-Cleaning-Data-Course-Project
This is the summary of process on the code snipets below

  - Download the dataset by using the file.download function if not existing in the working directory.
  - Read the activity and feature info by using the read.table function to allow txt files to be in data table format
  - Read both the training and test datasets, then filter the data using grep() function getting only the columns with "mean" and "std"
  - Read the activity and subject data for each dataset, then merge the column into the dataset
  - Change the field names of the dataset by the colnames() function using the name of the rows available in features
  - Fix the naming convention by using gsub() function to remove uneccesary elements such as parenthesis and also have proper           capitalization
  - Create a final data set which gets the average of the data per activityname and activityid/ subject.
  - write the file in .txt format by using the write.table function and name the file tidydata.txt
  

#SOURCE CODE


#Start by Downloading the zip file in the working directory
temp <- tempfile()
download.file("https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip",temp)
data <- read.table(unz(temp, "a1.dat"))
temp
fileURL <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileUrl, destfile = "./data/camera.csv", method = "curl")

library(data.table)


activity_labels <- read.table("./GETTING AND CLEANING DATASET/UCI HAR Dataset/activity_labels.txt", header = F)
features <- read.table("./GETTING AND CLEANING DATASET/UCI HAR Dataset/features.txt", header = F)
head(activity_labels)
head(features)

#count the number of row to ensure that the number of columns in X_set is the same
nrow(features)

##Getting test data

X_test <- read.table("./GETTING AND CLEANING DATASET/UCI HAR Dataset/test/X_test.txt", header = F)
y_test <- read.table("./GETTING AND CLEANING DATASET/UCI HAR Dataset/test/y_test.txt", header = F)
subject_test <- read.table("./GETTING AND CLEANING DATASET/UCI HAR Dataset/test/subject_test.txt", header = F)
head(X_test)

#ensure column is match with feature row
ncol(X_test)

head(y_test)
head(subject_test)

#change column name
colnames(activity_labels) <- c("activityid", "activityname")
head(activity_labels)
colnames(subject_test) <- "subid"
head(subject_test)
colnames(X_test) <- features[,2]
head(X_test)
colnames(y_test) <- "activityid"

testdata <- cbind(subject_test,X_test,y_test)
View(testdata)

##Getting Train Data

X_train <- read.table("./GETTING AND CLEANING DATASET/UCI HAR Dataset/train/X_train.txt", header = F)
y_train <- read.table("./GETTING AND CLEANING DATASET/UCI HAR Dataset/train/y_train.txt", header = F)
subject_train <- read.table("./GETTING AND CLEANING DATASET/UCI HAR Dataset/train/subject_train.txt", header = F)


colnames(subject_train) <- "subid"
head(subject_train)
colnames(X_train) <- features[,2]
head(X_train)
colnames(y_train) <- "activityid"
head(y_train)

#Trial
traindata <- cbind(subject_train,X_train, y_train)
View(traindata)

nrow(traindata)
nrow(testdata)
Subject <- rbind(subject_train, subject_test)

finaldata <- rbind(testdata, traindata)
nrow(finaldata)
View(finaldata)



colnames(finaldata) 
colNames <- colnames(finaldata)
colNames

#extract only mean and std measurements
library(dplyr)
finaldata2 <- finaldata[ , grep( "Mean|std|activity" , names( finaldata ) ) ]
head(finaldata2)
colnames(finaldata2)

#uses descriptive activity names
finaldata3 <- merge(finaldata2, activity_labels, by="activityid")
head(finaldata3)

#Label the data set with descriptive variable name
#removing of parenthesis
names(finaldata3) <- gsub("\\(|\\)", "", names(finaldata3))
colnames(finaldata3)
colnames(finaldata3) <- gsub("^t", "Time", names(finaldata3))
colnames(finaldata3)
colnames(finaldata3) <- gsub("Acc", "Acceleration", names(finaldata3))
colnames(finaldata3) <- gsub("[t]Body", "TimeBody", names(finaldata3))
colnames(finaldata3) <- gsub("^f", "Frequency", names(finaldata3))
colnames(finaldata3) <- gsub("BodyBody", "Body", names(finaldata3))
colnames(finaldata3) <- gsub("Mag", "Magnitude", names(finaldata3))
colnames(finaldata3) <- gsub("std", "Std", names(finaldata3))

#put space in between strings
colnames(finaldata3) <- gsub("([a-z])([A-Z])", "\\1 \\2", names(finaldata3))
colnames(finaldata3)
ncol(finaldata3)
finaldata3 <- merge(finaldata3, Subject, by="subid")

#get the average of each variable per activity name
finaldata4<- finaldata3 %>% group_by(activityid, activityname) %>% summarise_all(funs(mean))
View(finaldata4)

write.table(finaldata4,file="tidydata.txt")
