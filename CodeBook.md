##set working directory
setwd("c:/Users/Marc.Haserodt/desktop/Coursera/gcd")
##download course files
download.file("https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip",destfile="./DS.zip")
##unzip files
unzip(zipfile="DS.zip")

##load packages
library(data.table)
library(dplyr)
library(tidyr)

##create object fileSpot to prevent having to type out the file location all the time
fileSpot <- getwd()

##store data labels in variables
featureNames <- read.table("UCI HAR Dataset/features.txt")
activityNames <- read.table("UCI HAR Dataset/activity_labels.txt", header = FALSE)

##read all the training data
trainSubject <- read.table("UCI HAR Dataset/train/subject_train.txt", header = FALSE)
trainActivity <- read.table("UCI HAR Dataset/train/y_train.txt", header = FALSE)
trainFeatures <- read.table("UCI HAR Dataset/train/X_train.txt", header = FALSE)

##read all the test data
testSubject <- read.table("UCI HAR Dataset/test/subject_test.txt", header = FALSE)
testActivity <- read.table("UCI HAR Dataset/test/y_test.txt", header = FALSE)
testFeatures <- read.table("UCI HAR Dataset/test/X_test.txt", header = FALSE)

##merge test and training data
subject <- rbind(testSubject, trainSubject)
activity <- rbind(testActivity, trainActivity)
features <- rbind(testFeatures, trainFeatures)

##name columns
colnames(features) <- t(featureNames[2])

##merge activity, features, and subject
colnames(features) <- t(featureNames[2])
colnames(activity) <- "Activity"
colnames(subject) <- "Subject"
allData <- cbind(features,activity,subject)

##Part 2
##Get the columns that have Mean and Standard Deviation in them
meanDevColumns <- grep(".*Mean.*|.*Std.*", names(allData), ignore.case=TRUE)

## add activity and subject and double check with dim()
requiredColumns <- c(meanDevColumns, 562, 563)
dim(allData)

## extract requiredColumns and again double check with dim()
dataExtract <- allData[,requiredColumns]
dim(dataExtract)

##Part 3
##change from numeric to character data types and factor activity
dataExtract$Activity <- as.character(dataExtract$Activity)
for (i in 1:6){
dataExtract$Activity[dataExtract$Activity == i] <- as.character(activityLabels[i,2])
}

dataExtract$Activity <- as.factor(dataExtract$Activity)

##Part 4
##Look at the names...look at them!
names(dataExtract)

##That does not fit tidy data standards.  Those labels can be more descriptive and less acronymy
names(dataExtract)<-gsub("Acc", "Accelerometer", names(dataExtract))
names(dataExtract)<-gsub("angle", "Angle", names(dataExtract))
names(dataExtract)<-gsub("BodyBody", "Body", names(dataExtract))
names(dataExtract)<-gsub("gravity", "Gravity", names(dataExtract))
names(dataExtract)<-gsub("Gyro", "Gyroscope", names(dataExtract))
names(dataExtract)<-gsub("Mag", "Magnitude", names(dataExtract))
names(dataExtract)<-gsub("tBody", "TimeBody", names(dataExtract))
names(dataExtract)<-gsub("^f", "Frequency", names(dataExtract))
names(dataExtract)<-gsub("^t", "Time", names(dataExtract))
names(dataExtract)<-gsub("-freq()", "Frequency", names(dataExtract), ignore.case = TRUE)
names(dataExtract)<-gsub("-mean()", "Mean", names(dataExtract), ignore.case = TRUE)
names(dataExtract)<-gsub("-std()", "StandardDeviation", names(dataExtract), ignore.case = TRUE)

##Look again.  That's better!
names(dataExtract)


##Part 5
##Set the factor variable
dataExtract$Subject <- as.factor(dataExtract$Subject)
dataExtract <- data.table(dataExtract)

##Create tidy as a tidy dataset
tidy <- aggregate(. ~Subject + Activity, dataExtract, mean)
tidy <- tidy[order(tidy$Subject,tidy$Activity),]
write.table(tidy, file = "Tidy.txt", row.names = FALSE)