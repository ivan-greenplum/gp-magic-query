
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
Validate the data is in the system.

First login to the twitter database
```
psql twitter
```
After login you will be at the database prompt for the twitter database
```
twitter=#
```
Now run this query to count the rows of data in tweets table
```
select count(*) from tweets;
```
You will see this outputs:
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

## STEP 9: PL/Python Exercise 
Modify the function to return returns the first 20 characters of a tweet and CAPITALIZES the tweet if its "possibly_sensitive" according to the "possibly_sensitive" column and lower case it if its not sensitive according to that field.

EXAMPLE OUTPUT
```
twitter=# select capitalize_string(tweet_text, possibly_sensitive) from tweets where possibly_sensitive = 't' and lang='en' limit 10;
  capitalize_string
----------------------
 RT @0RUBENMARTINEZ0:
 RT @IAM_JOHNW: THERE
 RT @JOONSJINN: CHEA
 2 SATELLITES ON POSS
 @RELLYCOOPER WHO RAI
 RT @HUMORANDANIMALS:
 RT @KOKOBUTTZ: GIFT
 RT @APPE4L: AFTER A
 RT @WINGSFORX1: EVEN
 RT @WITH_YUGYEOM: YU
(10 rows)

twitter=# select capitalize_string(tweet_text, possibly_sensitive) from tweets where possibly_sensitive = 'f' and lang='
en' limit 10;
   capitalize_string
------------------------
 you don't have to sp
 .@nctq’s #2020tpr is
 "needing benefits is
 rt @mrfegaa: please
 rt @tintinresists: @
 rt @aewontnt: can we
 gms newest super cru
 rt @mialis79: never
 rt @jayyarrrrm_: nev
 rt @allovic87: he lo
(10 rows)

twitter=#
```

## STEP 10: Exporting results

Export the data into a csv file
```
COPY (SELECT possibly_sensitive, capitalize_string(tweet_text, possibly_sensitive) 
      FROM tweets 
      ORDER BY 1) 
TO '/home/gpadmin/tweetdatafile.csv' 
WITH CSV HEADER;
```

Get the file and have a look (windows command):
```
cp tweetdatafile.csv /mnt/c/Users/inovick/Downloads/
```

## Step 11: JSON Exercise 1
Filter the tweets for tweets with valid coordinates of tweets that have valid data and don't return a null json output.  
```
CREATE TABLE geotweets AS
SELECT *
FROM tweets
WHERE coordinates IS NOT NULL AND coordinates::text <> '' and coordinates::text <> 'null';
```

## Step 11: JSON Exercise 1
Print coordinates as text
```
select coordinates::text from geotweets;
```

## Step 12: JSON Exercise 2
Convert the coordinates to LAT/LONG and also return user_location field
EXAMPLE OUTPUT
```
              user_location               |   longitude   |   latitude
------------------------------------------+---------------+--------------
 "Louisville, KY"                         |      -85.7276 |      38.2361
 "Las Vegas, NV"                          |      -115.149 |      36.1675
 "Madrid"                                 |       -3.8647 |      40.3218
 "Karachi"                                |      -95.6368 |      29.6183
 "Edmonton, Alberta"                      |    -113.51612 |      53.5035
 "Chicago"                                |      -96.7255 |      17.0603
 "Paraguay"                               |  -57.54013443 | -25.23498314
 "La Costa, Argentina"                    |     -56.69164 |    -36.54087
 "Carson, CA"                             |    -118.28494 |     33.80762
 "Winnipeg, Manitoba"                     |   -97.1751082 |  49.78289497
 "Westminster, MA"                        |    -118.26711 |     34.04312
 "Williamsville, NY"                      |   -78.7489499 |     42.96233
 "Palermo, Italy"                         |   13.37007926 |  38.11773613
 "Saint Louis, MO"                        |   -90.3371889 |   38.6105426
 "Springfield, IL"                        |   -88.9521125 |   39.9213359
 "Detroit, MI"                            |   -83.2218731 |   42.4733688
 "Tampa, FL"                              |      -82.5923 |      28.0432
 "\u65e5\u672c \u5e83\u5cf6"              |  132.32191144 |  34.30085844
 "Catol\u00e9 do Rocha, Brasil"           |     -37.74584 |      -6.3468
 "Texas, Houston"                         |  -95.22071424 |   29.6193794
 "Goldsboro, NC"                          |       -77.978 |       35.382
 "Sedona, Arizona"                        | -111.76436906 |  34.77471729
 "South Beach, FL"                        |  -80.13085376 |  25.78115433
 "Chicago"                                |    -87.632496 |    41.883222
 "San Leandro, CA"                        |      -118.347 |      34.2061
 "San Francisco, CA"                      |  -122.4852507 |   37.8590937
 "Fairfax, VA"                            |   -77.3209555 |   38.6805859
```

## Step 13: JSON Exercise 3
Confirm online with google/chatgpt that the user_location matches the long/lat 
For example:
```
The approximate latitude and longitude coordinates for Las Vegas, Nevada are:
- Latitude: 36.1699° N
- Longitude: 115.1398° W
```

## Step 14: Install Postgis
Install postgis using gppkg as gpadmin

```
cp /mnt/c/Users/ivan/Downloads/postgis-3.3.2+pivotal.2.build.1-gp7-rhel8-x86_64.gppkg .
```

Install pre-req packages on Rocky 8
```
sudo dnf install libtiff
```

Install postgis
```
gppkg install postgis-3.3.2+pivotal.2.build.1-gp7-rhel8-x86_64.gppkg
```

Login to Twitter database and create the extension
```
CREATE EXTENSION IF NOT EXISTS postgis;
SELECT PostGIS_Version();
```

Output:
```
twitter=# SELECT PostGIS_Version();
            postgis_version
---------------------------------------
 3.3 USE_GEOS=1 USE_PROJ=1 USE_STATS=1
(1 row)
```

## Step 15: Load USSTATES data

Get the data files fron US Gov:
```
sudo dnf install wget
wget https://www2.census.gov/geo/tiger/GENZ2018/shp/cb_2018_us_state_500k.zip
unzip cb_2018_us_state_500k.zip
```
Load the data into the twitter database
```
shp2pgsql -s 4269 -D cb_2018_us_state_500k.shp usstates > usstates.sql
psql -f usstates.sql twitter
```
test in psql
```
select gid,name from usstates;
```

OUTPUT:
```
twitter=# select gid,name from usstates;
 gid |                     name
-----+----------------------------------------------
   2 | North Carolina
   3 | Oklahoma
   4 | Virginia
   6 | Louisiana
   7 | Michigan
   8 | Massachusetts
   9 | Idaho
  10 | Florida
  13 | New Mexico
```

## Step 16: calculate the area of each usstates and sort by size
use ST_Area on the geom column to get the output of area by state
```
 gid |                     name                     |         area
-----+----------------------------------------------+----------------------
   1 | Mississippi                                  |    11.88541662019452
   5 | West Virginia                                |    6.493879726220489
  11 | Nebraska                                     |   21.614456008090006
  12 | Washington                                   |    20.89963605560926
  14 | Puerto Rico                                  |    0.771304755414308
  15 | South Dakota                                 |   22.578331635133495
  17 | California                                   |    41.66827322569092
  20 | Pennsylvania                                 |    12.53583770321049
  23 | Utah                                         |   22.975116780908895
  25 | Wyoming                                      |   27.970714628094303
  26 | New York                                     |   14.006968575208102
  30 | Illinois                                     |   15.408210633680493
  31 | Vermont                                      |   2.7979535129125015
  35 | New Hampshire                                |    2.683566606943997
  36 | Arizona                                      |   28.919220031715934
```

You can see above its 11 units for Mississippi, but that is not corresponding to SQ Miles or SQ Meters

## Step 17: Convert the area to SQ Meters 
there are approximately 125,443,000,000 square meters in Mississippi.
there are approximately 380,831,000,000 square meters in Montana.

You can use a ST_Transform on the geom with SRID: 2163
Example:
```
 gid |     name      |      area_m2
-----+---------------+--------------------
   1 | Mississippi   | 123,563,796,126
   7 | Michigan      | 150,729,671,806
   8 | Massachusetts |  21,218,579,813
  32 | Montana       |  379,815,906,105
```

## Step 18: Add postgis geometry for geostweets
Add a new column and create a new table called **pgeotweets** by doing CTAS with a new column added by converting coordinates using ST_Transform(ST_GeomFromGeoJSON(coordinates), 4269)
```
twitter=# \d pgeotweets
                               Table "public.pgeotweets"
         Column          |            Type             | Collation | Nullable | Default
-------------------------+-----------------------------+-----------+----------+---------
 id                      | bigint                      |           |          |
 created_at              | timestamp without time zone |           |          |
 coordinates             | json                        |           |          |
 tweet_text              | text                        |           |          |
 full_text               | text                        |           |          |
 in_reply_to_status_id   | text                        |           |          |
 in_reply_to_user_id     | text                        |           |          |
 in_reply_to_screen_name | text                        |           |          |
 user_id                 | bigint                      |           |          |
 user_name               | text               9         |           |          |
 user_location           | text                        |           |          |
 hashtags                | json                        |           |          |
 user_mentions           | json                        |           |          |
 quote_count             | integer                     |           |          |
 reply_count             | integer                     |           |          |
 retweet_count           | integer                     |           |          |
 favorite_count          | integer                     |           |          |
 favorited               | boolean                     |           |          |
 retweeted               | boolean                     |           |          |
 possibly_sensitive      | boolean                     |           |          |
 lang                    | text                        |           |          |
 mentioned_user_ids      | bigint[]                    |           |          |
 geom                    | geometry                    |           |          |
Distributed randomly

```

## Step 19: Compare geom, lat long with coordinates lat lang
'''
SELECT ST_Y(geom) AS latitude, ST_X(geom) AS longitude, coordinates FROM pgeotweets;
'''

## Step 20: write a inner join query between pgeotweets and usstates

Now please do a join using ST Contains to print the tweets and their usstates

You need to do aa join on containment of a point in a polygon, hint:
```
 SELECT
    ST_Y(pgeotweets.geom) AS latitude,
    ST_X(pgeotweets.geom) AS longitude,
    pgeotweets.coordinates,
    pgeotweets.user_location,
    usstates.name
FROM
    pgeotweets
JOIN
    usstates
ON
    ST_Contains(usstates.geom, pgeotweets.geom);
```

Do some validation to ensure the returning data is as correct in the join
```
  latitude   |   longitude   |                        coordinates                         |         user_location          |         name
-------------+---------------+------------------------------------------------------------+--------------------------------+----------------------
     37.8414 |      -82.0089 | {"type":"Point","coordinates":[-82.0089,37.8414]}          | "Mount Gay-Shamrock, WV"       | West Virginia
     40.7727 |      -96.6807 | {"type":"Point","coordinates":[-96.6807,40.7727]}          | "Lincoln, NE"                  | Nebraska
     47.6046 |     -122.3308 | {"type":"Point","coordinates":[-122.3308,47.6046]}         | "Florida, USA"                 | Washington
```

Note: where is lat 47 long -122

## Step 21: Install pgvector
```
create extension vector
```

## Step 22: Install postgresml
Install Data Science Package
```
gppkg install DataSciencePython3.9-2.1.0-gp7-el8_x86_64.gppkg
```
Follow steps:
https://docs.vmware.com/en/VMware-Greenplum/7/greenplum-database/ref_guide-modules-postgresml.html
# Note this needs more details, give it a try

Create extension
```
create extension postgresml
```
