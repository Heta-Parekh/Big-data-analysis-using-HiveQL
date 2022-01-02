# Big-Data-Project
Cargurus Used Car Data Analysis using Apache Hadoop

# Platform Specifications
##### Cluster Version – Oracle Big Data Compute Edition
##### Number of Nodes – 3
##### Memory size – 180 GB
##### CPU – 12 OCPUs
##### CPU speed – 2.20 GHz
##### Storage – 957 GB

Below is the location of the CarGurus usedcar data that is used for this project. You can download
the data file from amazon S3 as follows:
```
$ wget -O CarData.zip http://usedcardataset.s3.amazonaws.com/archive.zip
```
Then, you need to unzip the files.
```
$ unzip CarData.zip
```
Run the following commands for creating directories:
```
$ hdfs dfs -mkdir GP2TermProject
$ hdfs dfs -mkdir GP2TermProject/tables
$ hdfs dfs -mkdir GP2TermProject/tables/cgdata
```
To view the files and directories created at HDFS use the following command:
```
$ hdfs dfs -ls GP2TermProject/tables
```
Run the following commands to put dictionaries into respective folders:
```
$ hdfs dfs -put /dev/shm/used_cars_data.csv GP2TermProject/tables/cgdata/
```
You can run the following commands to check the files are there:
```
$ hdfs dfs -ls GP2TermProject/tables/cgdata/
```

