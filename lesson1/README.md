# This is a structured lab that you can follow to use this course to learn sequentially

## STEP 1: Install Greenplum on a Linux VM or HOST

## STEP 2: Verify you can read and write data, gpstop, gpstart

## STEP 3: SQL EXERCISE 1
Write a SQL to Find the 5 most active twitter user names and how many tweets they have

## STEP 4: SQL EXERCISE 2
Write a SQL to Find the 5 most active twitter user names that tweet in spanish and how many tweets they have but only tweets in Spanish

## STEP 5: Install PL/Python in twitter database
create language plpython3u;

## STEP 6: PL/Python Exercise 1
create a function that returns the first 20 characters of a tweet and CAPITALIZES the tweet if its "possibly_sensitive" according to the "possibly_sensitive" column
Call the function and return the output for all tweets and output to a data file with the function return value and the "possibly_sensitive" column seperated as a CSV file called data.csv

## Step 7: JSON Exercise 1
Select the coordinates of tweets that have valid data and don't return a null json output.  Print coordinates as text

## Step 8: JSON Exercise 2
Select the non-null geocoordinates tweets returning user_location field, and then longitude and lattitude field.  Confirm online with google/chatgpt that the user_location matches the long/lat 

## Step 9: Postgis
Install postgis using gppkg as gpadmin
gppkg install PACKAGENAME
Login to Twitter database and create the extension
```
CREATE EXTENSION IF NOT EXISTS postgis;
SELECT PostGIS_Version();
```

## Step 10: Use Tiger geocoder to re-run the query and step 8
Now this time return 4 columns
user_location, tiger geocoded location, long and lat
Confirm they match when possible
