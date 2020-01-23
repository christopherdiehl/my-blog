+++
date = "2020-01-23T14:16:20+00:00"
title = "SQL Optimizations"
tags = ["sql","optimization", "Raleigh"]
keywords = ["sql","optimization", "Raleigh"]
draft = false
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
docker run --rm --network=host postgres psql -h localhost -p 5432 -U postgres -c '
    CREATE TABLE raleigh_police_incidents (
        OBJECTID int PRIMARY KEY,
        GlobalID TEXT NOT NULL DEFAULT '',
        case_number TEXT NOT NULL DEFAULT '',
        crime_category TEXT,
        crime_code varchar(5),
        crime_description TEXT,
        crime_type TEXT,
        reported_block_address TEXT,
        city_of_incident TEXT,
        city TEXT,
        district TEXT,
        reported_date datetime,
        reported_year int,
        reported_month int,
        reported_day int,
        reported_hour int,
        reported_dayofwk TEXT,
        latitude TEXT,
        longitude TEXT,
        agency TEXT,
        updated_date datettime
    );
'

docker run --rm --network=host -v `pwd`:/data postgres \
  psql -h localhost -p 5432 -U postgres -c \
    "\\copy raleigh_police_incidents FROM 'Raleigh_Police_Incidents_NIBRS.csv' WITH csv"

// get psql shell
docker exec -it postgres psql -U postgres
```