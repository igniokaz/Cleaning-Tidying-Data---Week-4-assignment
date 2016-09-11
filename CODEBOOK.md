#loading dplyr library
library(dplyr)
library(reshape2)

#downloading the data from the link into working directory
url<-"https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(url, "inputs.zip", method="curl")
unzip("inputs.zip")

#opening required files from the working directory
activitylabels<-read.table("UCI HAR Dataset/activity_labels.txt", stringsAsFactors = FALSE)
features<-read.table("UCI HAR Dataset/features.txt", stringsAsFactors = FALSE)
testx<-read.table("UCI HAR Dataset/test/X_test.txt", stringsAsFactors = FALSE)
testy<-read.table("UCI HAR Dataset/test/y_test.txt", stringsAsFactors = FALSE)
tests<-read.table("UCI HAR Dataset/test/subject_test.txt", stringsAsFactors = FALSE)
trainx<-read.table("UCI HAR Dataset/train/X_train.txt", stringsAsFactors = FALSE)
trainy<-read.table("UCI HAR Dataset/train/y_train.txt", stringsAsFactors = FALSE)
trains<-read.table("UCI HAR Dataset/train/subject_train.txt", stringsAsFactors = FALSE)

#extracting variable names as a character, and applying them to testX and trainX datasets as labels
u<-unlist(features[,2])
names(testx)<-paste(u)
names(trainx)<-paste(u)

#binding as columns activity id, person id and sensor data for test & train datasets separately
test<-cbind(testy, tests, testx)
train<-cbind(trainy, trains, trainx)

#adding meaningful labels for the first two columns of newly created datasets
names(test)[1]<-paste("ActID")
names(test)[2]<-paste("PersonID")
names(train)[1]<-paste("ActID")
names(train)[2]<-paste("PersonID")

#binding train and test datasets into one
full<-rbind(test, train)

#selecting columns from the full dataset for mean & standard deviation (any column containing "mean" or "std" respectively
tmp1<-full[,grep("mean", colnames(full))]
tmp2<-full[,grep("std", colnames(full))]

#combining activity id, person id and sensor data for means & std (this eliminates unnecessary columns from the full dataset)
full2<-cbind(full[,1:2],tmp1,tmp2)

#adding meaningful labels for activity labels dataset
names(activitylabels)[1]<-paste("ActID")
names(activitylabels)[2]<-paste("Activity")

#merging activity labels dataset with the latest version of full dataset (this adds activity description)
full3<-merge(activitylabels, full2)

#reshaping the dataset with melt & dcast functions, summarising the dataset as means per activity & person id combination
t<-melt(full3, id=c("ActID","Activity","PersonID"))
output<-dcast(t, ActID+Activity+PersonID~variable, mean)

#exporting the final dataset into working directory as txt file
write.table(output, "output.txt", row.names=FALSE)