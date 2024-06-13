# This is a structured lab that you can follow to use this course to learn sequentially

## STEP 1: Install Greenplum on a Linux VM or HOST
[installation](../../../blob/main/deploy.md)

## STEP 2: Verify you can read and write data, gpstop, gpstart
```
gpstop
gpstart
create table foo as select generate_series(1,100);
select * from foo;
```

## Step 3: Load the twitter data set
Clone the repo with the sample data locally.  This assumes you installed git in your linux environment during the installation.
```
sudo dnf install git
git clone https://github.com/greenplum-db/gp-magic-query.git
```

Create the database
run the data load
```
createdb twitter
psql twitter -f gp-magic-query/load-data-framework/twitter_data.sql
```
Validate the data is in the system
```
twitter=# select count(*) from tweets;
 count
--------
 110421
(1 row)
```

You can view the table structure like this:
```
twitter=# \d+ tweets
```

## STEP 4: SQL EXERCISE 1
Write a SQL to Find the 5 most active twitter user names and how many tweets they have

EXAMPLE
```
SELECT LEFT(user_name,20) as twitter_user, count(*) FROM
 tweets GROUP BY twitter_user ORDER by 2 desc;
```

## STEP 5: SQL EXERCISE 2
EXERCISE FOR YOU: 
Modify the previous SQL to Find the 5 most active twitter user names that tweet in spanish and how many tweets they have but only tweets in Spanish

```
     twitter_user     | count
----------------------+-------
 "\u064b"             |    14
 "."                  |    14
 "\ud83c\uddea\ud83c\ |    14
 "Ana"                |    12
 "Laura"              |    11
 "Luis"               |    11
 "Mar\u00eda"         |    11
 "Juan"               |    11
 "\ud835\udcdc\ud835\ |    11
 "dilme"              |    10
 "Yanet Rodriguez"    |     9
 "Sof\u00eda"         |     9
 "M"                  |     8
 "\ud835\udd78\ud835\ |     8
 "\ud835\udd77\ud835\ |     8
 "\ud83c\udf19"       |     8
 "Valentina"          |     8
 "Sandra"             |     7
 "Mario"              |     7
 "Oscar"              |     7
 "Eduardo"            |     7
 "Santiago"           |     7
 "\ud83c\uddfb\ud83c\ |     7
 "edgar bolivar"      |     7
 "\ud835\udc0b\ud835\ |     7
 "Paula"              |     7
 "Cristian"           |     7
```
## STEP 6: Install PL/Python in twitter database
```
create language plpython3u;
```

## STEP 7: Sample Pl/Python function
Create this function
```
CREATE OR REPLACE FUNCTION capitalize_string(input_text text)
RETURNS text AS $$
    return input_text.upper()
$$ LANGUAGE plpython3u;
```

Run the function
```
SELECT capitalize_string('hello world');
```

## STEP 8: PL/Python Exercise 
Modify the function to return only the first 20 characters of the string and then call it for each row in the tweets table.  Limit to 30 rows.

TIP: Write the function in a file called myfunc.sql and then run the file to import the function

EXPECTED SAMPLE OUTPUT:
```
twitter=# select capitalize_string(tweet_text) from tweets limit 10;
  capitalize_string
----------------------
 RT @RUBENCARABAJAL1:
 @REAL_GREG_BRADY @PO
 RT @GFIUZA_OFICIAL:
 RT @RICKSANCHEZZ01:
 RT @ELLENBEATRIZ_A:
 @EG_CT @JHOYT15 @CIA
 RT @MIKAIUST: A SIMP
 DESISTI DA CONTA, ES
 YOU DON'T HAVE TO SP
 RT @CLAUDIO38748087:
(10 rows)
```

## STEP 8: PL/Python Exercise 
Call this function for each row in tweets table and limit the output to 30 rows.

EXPECTED SAMPLE OUTPUT
```
--------------------------------------------------
 WALLACE E JULIA TEND
 RT @BERDINESTA: CREO
 RT @MANOELMSGONTIJO:
 RT @SAO_ANIME: 【-エクス
 RT @LUV_OYE: #ASIMRI
 RT @HIMSELFZOOT: IF
 RT @CATINFORMACIO: @
 @SANYUWU Y PQ LAS CO
 RT @PCHDOTCOM: IF YO
 ネイサンがTOYOTAの所属？というツイ
 RT @OHHEYYDEEE: SIS
 RT @WOLFFWALDO: TENE
 THE SAD TRUTH IS THA
 RT @JBFAN911: A LARG
 RT @TROLLFOOTBALL: S
 RAINBOW HOLOFOIL BUB
 YO VENGO A LEER EL P
 RT @MINBINHO: O MINH
 @THISISECW2020 @DEAN
 RT @SINGHLIONS: WE W
 RT @JESSYCAVITORIAA:
 @BEVOOHMATRON @71_HE
 RT @BEPPESALA: DOBBI
 RT @JALI_CAT: SO LET
 IPHONE 6S, DROPPED I
 @TEAMYOUTUBE I HAVE
 RT @LANNA2231: HOJE
 RT @CHANYEOLIT: 2001
 RT @SENTRYISVOID: LM
 EU
(30 rows)
```

## STEP 9: PL/Python Exercise 
Modify the function to return returns the first 20 characters of a tweet and CAPITALIZES the tweet if its "possibly_sensitive" according to the "possibly_sensitive" column and lower case it if its not sensitive according to that field.
```
```

## STEP 10: PL/Python Exercise 


Call the function and return the output for all tweets and output to a data file with the function return value and the "possibly_sensitive" column seperated as a CSV file called data.csv

```
psql -f myfunc.sql twitter
```
OUTPUT: 



## Step 10: JSON Exercise 1
Select the coordinates of tweets that have valid data and don't return a null json output.  Print coordinates as text
https://www.postgresql.org/docs/9.4/functions-json.html

## Step 11: JSON Exercise 2
Select the non-null geocoordinates tweets returning user_location field, and then longitude and lattitude field.  Confirm online with google/chatgpt that the user_location matches the long/lat 

## Step 12: Postgis
Install postgis using gppkg as gpadmin
gppkg install PACKAGENAME
Login to Twitter database and create the extension
```
CREATE EXTENSION IF NOT EXISTS postgis;
SELECT PostGIS_Version();
```

https://postgis.net/workshops/postgis-intro/geography.html

## Step 13: Load USSTATES data
```
wget https://www2.census.gov/geo/tiger/GENZ2018/shp/cb_2018_us_state_500k.zip
unzip cb_2018_us_state_500k.zip
shp2pgsql -s 4269 -D cb_2018_us_state_500k.shp usstates > usstates.sql
psql -f usstates.sql twitter
```
test in psql
```
select gid,name from usstates;
```
https://postgis.net/workshops/postgis-intro/loading_data.html

## Step 14: calculate the area of each usstates and sort by size
use ST_Area on the geom column
https://postgis.net/docs/ST_Area.html


## Step 15: write a inner join query between tweets and usstates
first make a copy of tweets table and filter out null geos and also add a column for geom data type using ST_GeomFromGeoJSON
you need to transform the coordinates system of one of the tables geometry to the same as the other
```
 ST_Transform(t.st_geomfromgeojson, 4269)
```
You need to do aa join on containment of a point in a polygon, hint:
```
FROM public.some_tweets t
JOIN public.usstates s ON ST_Contains
```
