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
Run the following HDFS command to provide permission to the files under the project folder.
**NOTE**: To make your beeline command work properly, use the following command.
```
$ hdfs dfs -chmod -R o+w GP2TermProject/
```
```
$ beeline
```
 Type the connect command as follows, and it should be given by the instructor:
 ```
 beeline> !connect
jdbc:hive2://bigdai-nov-bdcsce-1:2181,bigdai-nov-bdcsce-2:2181,bigdai-nov-bdcsce-3:2181/;serviceDi
scoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2?tez.queue.name=interactive
bdcsce_admin
```
**NOTE: If you see “CLOSED” in the above beeline shell prompt, it is not connected to Hive Server2.
Now you have to create your database with your username to separate your tables with other users. For
example, the user hparekh2 should run the following:**
```
0: jdbc:hive2://bigdai-nov-bdcsce-1:2181,bigd> create database if not exists hparekh2;
0: jdbc:hive2://bigdai-nov-bdcsce-1:2181,bigd> show databases;
0: jdbc:hive2://bigdai-nov-bdcsce-1:2181,bigd> use hparekh2;
```
In the beeline shell CLI, you need to copy and paste the following HiveQL code to create an external
table "cargurus_usedcars".
```
DROP TABLE IF EXISTS cargurus_usedcars;
CREATE EXTERNAL TABLE IF NOT EXISTS cargurus_usedcars(vin string, back_legroom string, bed string,
bed_height string, bed_length string, body_type string, cabin string, city string, city_fuel_economy
float, combine_fuel_economy int, daysonmarket int, dealer_zip string, description string,
engine_cylinders string, engine_displacement float, engine_type string, exterior_color string, fleet
string, frame_damaged string, franchise_dealer string, franchise_make string, front_legroom string,
fuel_tank_volume string, fuel_type string, has_accidents string, height string, highway_fuel_economy
float, horsepower int, interior_color string, iscab string, is_certified string, is_cpo string, is_new
boolean, is_oemcpo string, latitude double, length string, listed_date date, listing_color string,
listing_id bigint, longitude double, main_picture_url string, major_options string, make_name string,
maximum_seating string, mileage float, model_name string, owner_count int, power string, price float,
salvage string, savings_amount int, seller_rating float, sp_id bigint, sp_name string, theft_title string,
torque string, transmission string, transmission_display string, trimid string, trim_name string,
vehicle_damage_category string, wheel_system string, wheel_system_display string, wheelbase
string, width string, year int) ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES ("separatorChar" = "," , "quoteChar" = "\"") STORED AS TEXTFILE LOCATION
'/user/hparekh2/GP2TermProject/tables/cgdata/' TBLPROPERTIES ('skip.header.line.count' = '1');
```
