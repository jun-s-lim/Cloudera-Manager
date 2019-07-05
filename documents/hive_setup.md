# Uploading Data to HDFS For Storage and Analysis

1. upload data to hdfs
```bash
# make directories
hdfs dfs -mkdir /user/training/yelp
hdfs dfs -mkdir /user/training/yelp/business
hdfs dfs -mkdir /user/training/yelp/review
hdfs dfs -mkdir /user/training/yelp/users
hdfs dfs -mkdir /user/training/yelp/tip
hdfs dfs -mkdir /user/training/yelp/checkin

# put files
hdfs dfs -put business.json /user/training/yelp/business
hdfs dfs -put review.json /user/training/yelp/review
hdfs dfs -put users.json /user/training/yelp/users
hdfs dfs -put tip.json /user/training/yelp/tip
hdfs dfs -put checkin.json /user/training/yelp/checkin
```

# Hive SerDe library Configuration

1. download jar file
```
wget -O json-serde-1.3.8-jar-with-dependencies.jar  \
www.congiu.net/hive-json-serde/1.3.8/cdh5/json-serde-1.3.8-jar-with-dependencies.jar

wget -O json-udf-1.3.8-jar-with-dependencies.jar  \
www.congiu.net/hive-json-serde/1.3.8/cdh5/json-udf-1.3.8-jar-with-dependencies.jar

ln -s /opt/cloudera/parcels/CDH-5.15.2-1.cdh5.15.2.p0.3/lib/hive /usr/lib/hive
sudo cp *.jar /usr/lib/hive
```
2. deploy jar file
```text
#외부 Jar 등록
1. In the Cloudera Manager Admin Console, go to the Hive service.
2. Click the Configuration tab.
3. Under Filters, click "Hive (Service-Wide)" scope.
4. Click the Advanced category.
5. In the panel on the right, locate the "Hive Service Advanced Configuration Snippet (Safety Value) for hive-site.xml", click the plus sign (+) to the right of it, and enter the following information:
6. In the Name field, enter the "hive.reloadable.aux.jars.path" property.
7. In the Value field, enter the path "/usr/lib/hive/lib/json-serde-1.3.8-jar-with-dependencies.jar, /usr/lib/hive/lib/json-udf-1.3.8-jar-with-dependencies.jar" where you copied the JAR file to in Step 1.
8. In the Description field, enter the property description. For example, Path to Hive UDF JAR files.
9. Click Save Changes.
10. Redeploy the Hive client configuration. In the Cloudera Manager Admin Console, go to the Hive service.
From the Actions menu at the top right of the service page, select Deploy Client Configuration.
Click "Deploy Client Configuration".
11. Restart the Hive service.
```

# Creating Tables in HIVE

## Create Restaurants Table

1. add jars
```sql
ADD JAR json-serde-1.3.8-jar-with-dependencies.jar;
ADD JAR json-udf-1.3.8-jar-with-dependencies.jar;
```
2. create initial BUSINESS table (with all categories and attributes included)
```sql
CREATE EXTERNAL TABLE business4 (
address string,
business_id string,
categories array<string>,
city string,
hours struct<friday:string, monday:string, saturday:string, sunday:string, thursday:string,
tuesday:string, wednesday:string>,
is_open int,
latitude double,
longitude double,
name string,
neighborhood string,
postal_code string,
review_count int,
stars double,
state string,
Attributes struct<
Accepts_Insurance:boolean,
Ages_Allowed:string,
Alcohol:string,
Bike_Parking:boolean,
Business_Accepts_Bitcoin:boolean,
Business_Accepts_Credit_Cards:boolean,
By_Appointment_Only:boolean,
Byob:boolean,
BYOB_Corkage:string,
Caters:boolean,
Coat_Check:boolean,
Corkage:boolean,
Dogs_Allowed:boolean,
Drive_Thru:boolean,
Good_For_Dancing:boolean,
Good_For_Kids:boolean,
Happy_Hour:boolean,
Has_TV:boolean,
Noise_Level:string,
Open24Hours:boolean,
Outdoor_Seating:boolean,
Restaurants_Attire:string,
Restaurants_Counter_Service:boolean,
Restaurants_Delivery:boolean,
Restaurants_Good_For_Groups:boolean,
Restaurants_Reservations:boolean,
Restaurants_Table_Service:boolean,
Restaurants_Take_Out:boolean,
Smoking:string,
WheelchairAccessible:boolean,
WiFi:string,
Ambience:struct<
Casual:boolean,
Classy:boolean,
Divey:boolean,
Hipster:boolean,
Intimate:boolean,
Romantic:boolean,
Touristy:boolean,
Trendy:boolean,
Upscale:boolean>,
BestNights:struct<
Friday1:boolean,
Monday1:boolean,
Saturday1:boolean,
Sunday1:boolean,
Thursday1:boolean,
Tuesday1:boolean,
Wednesday1:boolean>,
BusinessParking:struct<
Garage:boolean,
Lot:boolean,
Street:boolean,
Valet:boolean,
Validated:boolean>,
DietaryRestrictions:struct<
Dairy_Free:boolean,
Gluten_Free:boolean,
Halal:boolean,
Kosher:boolean,
Soy_Free:boolean,
Vegan:boolean,
Vegetarian:boolean>,
GoodForMeal:struct<
Breakfast:boolean,
Brunch:boolean,
Dessert:boolean,
Dinner:boolean,
Latenight:boolean,
Lunch:boolean>,
HairSpecializesIn:struct<
Africanamerican:boolean,
Asian:boolean,
Coloring:boolean,
Curly:boolean,
Extensions:boolean,
Kids:boolean,
Perms:boolean,
Straightperms:boolean>,
Music:struct<
BackgroundMusic:boolean,
Dj:boolean,
Jukebox:boolean,
Karaoke:boolean,
Live:boolean,
NoMusic:boolean,
Video:boolean>,
restaurantspricerange2:int>)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
STORED AS TEXTFILE
LOCATION '/user/training/yelp/business';
```
```sql
SELECT count(business_id) FROM business4;
```
```text
174567
```
```sql
-- create table for impala query using CTAS and saving format as parquet
CREATE TABLE biz4
STORED AS PARQUET
AS SELECT * FROM business4;
```
```sql
SELECT * FROM biz4 limit 1;
```
```text
4855 E Warner Rd, Ste B9	FYWN1wneV18bWNgQjJ2GNg	["Dentists","General Dentistry","Health & Medical","Oral Surgeons","Cosmetic Dentists","Orthodontists"]	Ahwatukee	{"friday":"7:30-17:00","monday":"7:30-17:00","saturday":null,"sunday":null,"thursday":"7:30-17:00","tuesday":"7:30-17:00","wednesday":"7:30-17:00"}133.3306902	-111.9785992	Dental by Design		85044	22	4.0	AZ	{"accepts_insurance":null,"ages_allowed":null,"alcohol":null,"bike_parking":null,"business_accepts_bitcoin":null,"business_accepts_credit_cards":null,"by_appointment_only":null,"byob":null,"byob_corkage":null,"caters":null,"coat_check":null,"corkage":null,"dogs_allowed":null,"drive_thru":null,"good_for_dancing":null,"good_for_kids":null,"happy_hour":null,"has_tv":null,"noise_level":null,"open24hours":null,"outdoor_seating":null,"restaurants_attire":null,"restaurants_counter_service":null,"restaurants_delivery":null,"restaurants_good_for_groups":null,"restaurants_reservations":null,"restaurants_table_service":null,"restaurants_take_out":null,"smoking":null,"wheelchairaccessible":null,"wifi":null,"ambience":null,"bestnights":null,"businessparking":null,"dietaryrestrictions":null,"goodformeal":null,"hairspecializesin":null,"music":null,"restaurantspricerange2":null}
```
3. Create EXPLODED table with flattened categories
```sql
CREATE TABLE exploded
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe' STORED AS TEXTFILE
LOCATION '/user/training/yelp/business/exploded'
AS
SELECT * FROM business4 LATERAL VIEW explode(categories) c AS cat_exploded;
```
```sql
SELECT * FROM exploded LIMIT 1;
```
```text
4855 E Warner Rd, Ste B9	FYWN1wneV18bWNgQjJ2GNg	["Dentists","General Dentistry","Health & Medical","Oral Surgeons","Cosmetic Dentists","Orthodontists"]	Ahwatukee	{"friday":"7:30-17:00","monday":"7:30-17:00","saturday":null,"sunday":null,"thursday":"7:30-17:00","tuesday":"7:30-17:00","wednesday":"7:30-17:00"}133.3306902	-111.9785992	Dental by Design		85044	22	4.0	AZ	{"accepts_insurance":null,"ages_allowed":null,"alcohol":null,"bike_parking":null,"business_accepts_bitcoin":null,"business_accepts_credit_cards":null,"by_appointment_only":null,"byob":null,"byob_corkage":null,"caters":null,"coat_check":null,"corkage":null,"dogs_allowed":null,"drive_thru":null,"good_for_dancing":null,"good_for_kids":null,"happy_hour":null,"has_tv":null,"noise_level":null,"open24hours":null,"outdoor_seating":null,"restaurants_attire":null,"restaurants_counter_service":null,"restaurants_delivery":null,"restaurants_good_for_groups":null,"restaurants_reservations":null,"restaurants_table_service":null,"restaurants_take_out":null,"smoking":null,"wheelchairaccessible":null,"wifi":null,"ambience":null,"bestnights":null,"businessparking":null,"dietaryrestrictions":null,"goodformeal":null,"hairspecializesin":null,"music":null,"restaurantspricerange2":null}	Dentists
Time taken: 0.092 seconds, Fetched: 1 row(s)
```
```sql
SELECT count (DISTINCT business_id) number_businesses FROM exploded;
```
```text
174067
```
```sql
SELECT count (business_id) number_restaurants FROM exploded WHERE cat_exploded="Restaurants";
```
```text
OK
54618
```
4. create RESTAURANTS table
```sql
CREATE TABLE restaurants
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe' STORED AS TEXTFILE
LOCATION '/user/mboldin/yelp/business/restaurants'
AS
SELECT * FROM exploded WHERE cat_exploded="Restaurants";
```
```sql
SELECT name, review_count, stars, cat_exploded category FROM restaurants LIMIT 5;
```
```text
Brick House Tavern + Tap	116	3.5	Restaurants
Messina	5	4.0	Restaurants
East Coast Coffee	3	4.5	Restaurants
Showmars Government Center	7	3.5	Restaurants
Alize Catering	12	3.0	Restaurants
```
```sql
SELECT name, attributes.ambience.romantic FROM restaurants LIMIT 5;
```
```text
Brick House Tavern + Tap	false
Messina	false
East Coast Coffee	NULL
Showmars Government Center	false
Alize Catering	true
```
```sql
SELECT name, state, city, attributes.ambience.romantic romantic FROM restaurants WHERE attributes.ambience.romantic = true LIMIT 10;
```
```text
Alize Catering	ON	Toronto	true
Fraticelli's Authentic Italian Grill	ON	Richmond Hill	true
Florentia	ON	Toronto	true
Cafe Sam	PA	Pittsburgh	true
87 West 2	OH	Westlake	true
The Range Steakhouse	NV	Las Vegas	true
Buddha Thai Bistro	NV	Las Vegas	true
Vin Santo	WI	Middleton	true
L'Amore Italian Restaurant	AZ	Phoenix	true
Julia Baker Confections	AZ	Phoenix	true
```

## Create Review Table

1. Create Review Table
```sql
CREATE EXTERNAL TABLE review ( business_id string,
cool int,
review_date string,
funny int, review_id string, stars int,
text string, useful int, user_id string)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe' STORED AS TEXTFILE
LOCATION '/user/training/yelp/review';
```
```sql
SELECT * FROM review LIMIT 1;
```
```text
0W4lkclzZThpx3V65bVgig	0	NULL	0	v0i_UHJMo_hPBq9bxWvW4w	5	Love the staff, love the meat, love the place. Prepare for a long line around lunch or dinner hours.

They ask you how you want you meat, lean or something maybe, I can't remember. Just say you don't want it too fatty.

Get a half sour pickle and a hot pepper. Hand cut french fries too.	0	bv2nCi5Qv5vroFiqKGopiw
```
```sql
SELECT count(*) FROM review;
```
```text
5261669
```
2. Create Review_Filtered Table
```sql
CREATE TABLE review_filtered
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe' STORED AS TEXTFILE
LOCATION '/user/training/yelp/review_filtered' AS
SELECT re.business_id, r.stars, r.user_id FROM review r JOIN restaurants re
ON r.business_id = re.business_id;
```
```sql
SELECT count(*) FROM review_filtered;
```
```text
3221419
```

## Create Users Table

1. Create Users Table
```sql
CREATE EXTERNAL TABLE users (
average_stars double,
compliment_cool int,
compliment_cute int,
compliment_funny int,
compliment_hot int,
compliment_list int,
compliment_more int,
compliment_note int,
compliment_photos int,
compliment_plain int,
compliment_profile int,
compliment_writer int,
cool int,
elite array<int>,
fans int,
friends array<string>,
funny int,
name string,
review_count int,
useful int,
user_id string,
yelping_since string)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
LOCATION '/user/training/yelp/users';
```
```sql
SELECT * FROM users limit 1;
```
```text
4.67	0	0	0	0	0	0	0	0	1	0	0	0	[]	0["cvVMmlU1ouS3I5fhutaryQ","nj6UZ8tdGo8YJ9lUMTVWNw","RTtdEVhAmeWqCSp0IgJ99w","t3UKA1sl4e6LY_xsjuvI0A","s057_BvOfnKNvQquJf7VNg","VYrdepCgdzJ4WaxP7dBGpg","XXLSk6sQQDyr3dZ4zE-O0g","Py8ThfExQaXF2Woqr7kWUw","233YNvzVtZ1ObkaNkUzNIw","L6iE9NpmHHJQTk0JQlRlSA","Y7XTMgZ_q5Bj5f9KhK1R4Q"]	0	Johnny	8	0	oMy_rEb0UBEmMlu-zcxnoQ	2014-11-03
```
```sql
SELECT count(distinct user_id) FROM users;
```
```text
1326101
```
2. Create Elite Users Table
```sql
CREATE TABLE elite_users
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe' STORED AS TEXTFILE
LOCATION '/user/training/yelp/users/elite'
AS
SELECT * FROM users LATERAL VIEW explode(elite) c AS elite_year;
```

## Create Tip Table

1. Create Tip Table
```sql
CREATE EXTERNAL TABLE tip (
text string,
date_tip string,
likes int,
business_id string,
user_id string)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe' STORED AS TEXTFILE
LOCATION '/user/mboldin/yelp/tip';
```

# Run Hive on Spark

1. 하이브 Shell에서 엔진 세팅
```text
set hive.execution.engine=spark;
```
2. 쿼리 실행하기
```sql
SELECT count (DISTINCT business_id) number_businesses FROM exploded;
```
```text
174067
```

##  에러 발생할 경우

1. 하이브 디버그 모드로 실행
```bash
hive --hiveconf hive.root.logger=DEBUG,console
```
2. 쿼리 실행하기
3. `NoClassDefFoundError com.apache.hadoop.fs.FSDataInputStream` 에러 발생할 경우 아래 진행
4. 모든 노드에 아래처럼 spark env에 hadoop classpath 추가 [(참고문서)](https://www.cloudera.com/documentation/data-science-workbench/1-1-x/topics/cdsw_spark_configuration.html)
```bash
sudo vi /etc/spark/conf/spark-env.sh
```
```bash
if [[ -d $SPARK_HOME/python ]]
then
    for i in
    do
        SPARK_DIST_CLASSPATH=${SPARK_DIST_CLASSPATH}:$i
    done
fi

SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:$SPARK_LIBRARY_PATH/spark-assembly.jar"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:/usr/lib/hadoop/lib/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:/usr/lib/hadoop/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:/usr/lib/hadoop-hdfs/lib/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:/usr/lib/hadoop-hdfs/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:/usr/lib/hadoop-mapreduce/lib/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:/usr/lib/hadoop-mapreduce/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:/usr/lib/hadoop-yarn/lib/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:/usr/lib/hadoop-yarn/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:/usr/lib/hive/lib/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:/usr/lib/flume-ng/lib/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:/usr/lib/parquet/lib/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:/usr/lib/avro/*"
SPARK_DIST_CLASSPATH="$SPARK_DIST_CLASSPATH:$(hadoop classpath)" # <- 추가하기
```
5. 다시 hive에서 쿼리 실행하기
```text
set hive.execution.engine=spark;
```
```sql
SELECT count (DISTINCT business_id) number_businesses FROM exploded;
```
```text
174067
```