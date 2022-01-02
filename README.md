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
Then, in the beeline shell, you need to check if the table “cargurus_usedcars” is shown:
```
0: jdbc:hive2://bigdai-nov-bdcsce-1:2181,bigd> show tables;
```
Next you can query the content of the cargurus_usedcars table:
```
select vin, body_type, city, latitude, longitude, make_name, maximum_seating, mileage, model_name,
price, seller_rating, transmission, trim_name, year from cargurus_usedcars limit 20;
```
## 1. The below command will create a table based on the make and model of the car which has more seller ratings in the past 5 years.
Copy and paste the following Hive code to Beeline shell to create a table car_ratings:
```
CREATE EXTERNAL TABLE IF NOT EXISTS car_rating (
make_name string, model_name string,year int, seller_rating string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE LOCATION '/user/hparekh2/GP2TermProject/tables/car_rating/';

INSERT OVERWRITE TABLE car_rating
Select make_name, model_name, year, AVG(seller_rating)
FROM cargurus_usedcars
WHERE year >= 2015 and year <= 2020
GROUP BY make_name, model_name, year;
```
## 2. The below command provides N-gram sentiment analysis of top seller rating cars.
```
Create table bigram_analysis(new_ar array<struct<ngram:array<string>, estfrequency:double>>);
INSERT OVERWRITE TABLE bigram_analysis
Select context_ngrams(sentences(lower(description)), array(null,null),100)as bigram
FROM cargurus_usedcars
WHERE make_name IN ('Lotus','Bentley', 'Ferrari', 'Karma', 'Lamborghini', 'Rolls-Royce', 'Porsche',
'McLaren', 'Aston Martin', 'smart', 'MINI', 'Jaguar', 'Tesla', 'Lexus', 'Audi', 'Land Rover', 'Mercedes')
AND year >= 2015;
Create table frequency_bigram (ngram string, estfrequency double)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE LOCATION '/user/hparekh2/GP2TermProject/tables/ngramAnalysis/';
Insert overwrite table frequency_bigram
Select concat(X.ngram[0], ' ',X.ngram[1]) as bigram, X.estfrequency
From bigram_analysis LATERAL VIEW explode(new_ar) Z as X;
```
Now you can query the content of the table:
```
select * from frequency_bigram limit 10;
```
## 3. 3. The below query will create a table based on the inventory of cars based on body type and price range for the past 5 years.
Copy and paste the following Hive code to Beeline shell to create a table carsPriceRange :
This table will analyse data based on the price range.
```
CREATE TABLE carsPriceRange AS select body_type, price , case when price > 0 and price < 10000 then
'0 - 10000' when price > 10000 and price < 20000 then '10000 - 20000' when price > 20000 and price <
30000 then '20000 - 30000' when price > 30000 and price < 40000 then '30000 - 40000' when price >
40000 and price < 50000 then '40000 - 50000' when price > 50000 and price < 60000 then '50000 -
60000' when price > 60000 and price < 70000 then '60000 - 70000' when price > 70000 and price <
80000 then '70000 - 80000' when price > 80000 and price < 90000 then '80000 - 90000' else '90000 -
above' end as price_range from cargurus_usedcars where body_type != '' and year >= 2015 and year
<= 2020;
```
Now you can query the content of the table:
```
select * from carsPriceRange limit 10;
```
Once the carsPriceRange table has been created we will group the data by body type and price range to
get the final result.
```
CREATE EXTERNAL TABLE IF NOT EXISTS CarsPriceRangeByBodyType(price_range string, body_type
string,Car_count string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' STORED AS TEXTFILE
LOCATION '/user/hparekh2/GP2TermProject/tables/CarsPriceRangeByBodyType/';

INSERT OVERWRITE TABLE CarsPriceRangeByBodyType select
price_range,body_type,count(price_range) from carsPriceRange group by price_range,body_type
order by price_range;
```
Now you can query the content of the table:
```
select * from CarsPriceRangeByBodyType limit 10;
```


