+++
date = "2020-01-23T14:16:20+00:00"
title = "SQL Optimizations"
tags = ["sql","optimization", "Raleigh"]
keywords = ["sql","optimization", "Raleigh"]
draft = true
+++

## SQL Optimizations on a Raleigh Crime Dataset.

In my opinion, SQL is easy to get working initially, but optimizing SQL is a whole different beast. Thankfully, I recently had the chance to optimize a whole bunch of SQL for one of our critical platforms resulting in an over 300% reduction in execution time. While I can't promise the same result for anyone following this article, I do outline some of my learnings below using the most recent Raleigh Crime dataset.

### Requirements
- Download the dataset here: http://data-ral.opendata.arcgis.com/datasets/raleigh-police-incidents-nibrs
- Docker
- Experience with command line / SQL

### Setup
Let's startup the database and load the csv using the commands below:
```
docker run --name postgres -p 5432:5432 -d postgres # start the database in a docker container

docker cp Raleigh_Police_Incidents_NIBRS.csv postgres:/police_incidents.csv

// get psql shell
docker exec -it postgres psql -U postgres

CREATE TABLE raleigh_police_incidents (
        OBJECTID int PRIMARY KEY,
        GlobalID TEXT NOT NULL DEFAULT '',
        case_number TEXT DEFAULT '',
        crime_category TEXT,
        crime_code varchar(5),
        crime_description TEXT,
        crime_type TEXT,
        reported_block_address TEXT,
        city_of_incident TEXT,
        city TEXT,
        district TEXT,
        reported_date timestamp,
        reported_year int,
        reported_month int,
        reported_day int,
        reported_hour int,
        reported_dayofwk TEXT,
        latitude TEXT,
        longitude TEXT,
        agency TEXT,
        updated_date timestamp
    );
    
\copy raleigh_police_incidents FROM 'police_incidents.csv' WITH  CSV HEADER DELIMITER E'\t';
```

# Exploring the data
To familarize yourself with the data, try to find the following:

1. Find how many incidents don't possess a case number 
1. The number of distinct crime types
1. Earliest reported_date in the data set.
1. Crimes frequency by day of week
1. Crimes that happen at night
1. Crimes that happen during the day (8AM to 8 PM)
## Answers to Exploring the Data

1. 60515 with query: `SELECT COUNT(*) FROM raleigh_police_incidents WHERE case_number is null;`
1. 3 not including the incidents without a specified crime_type 
```
SELECT COUNT(*) FROM  (SELECT DISTINCT crime_type  FROM raleigh_police_incidents WHERE crime_type IS NOT NULL GROUP BY crime_type) crime_types;
```
1. 2014-06-1 04:15:00 `SELECT MIN(reported_date) from raleigh_police_incidents ;`
1. 
```
 count | dayofweek 
-------+-----------
 32011 |         0
 35514 |         1
 34620 |         2
 34515 |         3
 33850 |         4
 34820 |         5
 32900 |         6

SELECT COUNT(*), EXTRACT(DOW from reported_date ) as dayOfWeek FROM raleigh_police_incidents GROUP BY dayOfWeek; 
```
1. 129256 `SELECT COUNT(*) FROM (select extract(hour from reported_date) as rhour from raleigh_police_incidents WHERE extract(hour from reported_date) > 7 AND extract(hour from reported_date) < 21) t;`
1. 108974  `SELECT COUNT(*) FROM (select extract(hour from reported_date) as rhour from raleigh_police_incidents WHERE extract(hour from reported_date) < 8 OR extract(hour from reported_date) > 20) t;`