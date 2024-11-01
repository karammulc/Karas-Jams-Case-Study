# Kara's-Jams

## Table of Contents
- [Introduction](#introduction)
- [Documentation](#documentation)
- [Challenges](#challenges)
- [Key Findings](#key-findings)
- [Dashboard](#Dashboard)

  
# Introduction

###  StreamTunes-Case-Study
Hello! This project is especially personal and meaningful to me. Throughout my life, music has offered solace and fun opportunities for connection. 
Instead of addressing a business problem, this project will explore my Spotify data and personal listening patterns. 


#### About the dataset: 
- The data used for this project is my own historical spotify data 
- Account Data for any spotify user is available within account settings
- The historical data contains just over a year of data between 5/31/23 to 6/01/24
- Before cleaning, this dataset held 38,489 observations

## Tools Used: 
- BigQuery (SQL)
- Looker Studio
- Google Sheets
- Python


### Objectives: 
- Explore the patterns within my relationship to music
- Play with new tools
- Offer the opportunity for you to get to know me better
  
Let's go!

### Documentation
[Cleaning,Manipulation & Visualization Markdown](https://github.com/karammulc/Karas-Jams-Case-Study/blob/main/Cleaning%20%26%20Exploration.md)


Dashboard Version 1
![*Dashboard Version 1*](https://github.com/karammulc/Karas-Jams-Case-Study/blob/main/Images/Version%202.png) 

Dashboard Version 2
![*Dashboard Version 2 p1*](https://github.com/karammulc/Karas-Jams-Case-Study/blob/main/Images/Version%201%20P1.jpg) 
![*Dashboard Version 2 p1*](https://github.com/karammulc/Karas-Jams-Case-Study/blob/main/Images/Version%201%20P1.jpg) 
![*Dashboard Version 2 p1*](https://github.com/karammulc/Karas-Jams-Case-Study/blob/main/Images/Version%201%20P3.jpg) 

Initial Dashboard Exploration

![*Kara's Jams Dashboard page 1*](https://github.com/karammulc/Karas-Jams/blob/main/Images/Dashboard%20PG%201.png) 
![*Kara's Jams Dashboard page 2*](https://github.com/karammulc/Karas-Jams/blob/main/Images/Dashboard%20PG%202.jpg)

Second Iteration

# SQL Cleaning & Exploration

The data used for this project was requested directly from Spotify within personal account settings.
Four separate json files were received but were combined prior to upload by utilizing Python. 
This process resulted in one large CSV. 


```Python
import json
import csv

# Open the JSON files and read the data
with open('StreamingHistory_music_0.json') as file:
    data_0 = json.load(file)
with open('StreamingHistory_music_1.json') as file:
    data_1 = json.load(file)
with open('StreamingHistory_music_2.json') as file:
    data_2 = json.load(file)
with open('StreamingHistory_music_3.json') as file:
    data_3 = json.load(file)

# Combine the data into a single list
data = data_0 + data_1 + data_2 + data_3

# Create a CSV file and write the data
with open('StreamingHistory_music.csv', 'w', newline='') as file:
    writer = csv.writer(file)

    # Write the header row
    writer.writerow(data[0].keys())

    # Write the data rows
    for row in data:
        writer.writerow(row.values())
```

#### Duplicate check

```SQL
SELECT
    endtime,
    artistName,
    trackName,
    msplayed,
    COUNT(*) AS count
FROM
    `bamboo-life-418613.karasdata.jams`
GROUP BY
    endtime,
    artistName,
    trackName
    msplayed
HAVING
    COUNT(*) > 1;
```

#### No Duplicates found

#### Checking for Null Values

```SQL
SELECT *
FROM `karasdata.jams`
WHERE endtime IS NULL
   OR artistName IS NULL
   OR trackName IS NULL
   OR msplayed IS NULL
   OR endTime IS NULL;
```

#### No nulls found

- The first step in manipulation is to add various time-based columns. 
- The addition of secplayed and minplayed is for easier readability than milliseconds.
- Time of day and day of week are to be added for a more extensive temporal analysis. 
- In updating all rows for new columns; a WHERE TRUE statement is used as bigquery requires WHERE statements while using UPDATE.


#### Adding secplayed column 
```SQL
ALTER TABLE karasdata.jams
ADD COLUMN secsplayed FLOAT64;
```

#### Updating secplayed column 
```SQL
UPDATE karasdata.jams
SET secsplayed = msplayed / 1000
WHERE TRUE;
```

#### Adding minplayed column
```SQL
ALTER TABLE karasdata.jams
ADD COLUMN minsplayed FLOAT64;

```
#### Update minplayed column
```SQL
UPDATE karasdata.jams
SET minsplayed = secsplayed / 60 
WHERE TRUE;

```

#### Adding a day of the week column 
```SQL

ALTER TABLE karasdata.jams
ADD COLUMN day_of_week STRING;

```
#### Updating day of the week column 
```SQL
UPDATE karasdata.jams
SET day_of_week = FORMAT_TIMESTAMP('%A', TIMESTAMP(endtime))
WHERE TRUE;
```
#### Adding time of day column 
```SQL
ALTER TABLE karasdata.jams
ADD COLUMN time_of_day STRING;
```

#### Updating time of day column
```SQL
UPDATE karasdata.jams
SET time_of_day = (
  CASE
    WHEN EXTRACT(HOUR FROM TIMESTAMP(endtime)) BETWEEN 9 AND 15 THEN 'morning'
    WHEN EXTRACT(HOUR FROM TIMESTAMP(endtime)) BETWEEN 16 AND 21 THEN 'afternoon'
    ELSE 'night' 
  END
);

```


```SQL
DELETE FROM karasjams.jams
WHERE msplayed = 0;
```

Upon exploration, I found that some records had 0 ms played but the records still existed.
I removed these records during my cleaning. Prior to deletion, a backup was exported to a Google Cloud storage bucket and exported for local storage.

#### This statement removed 962 rows of 38489 from jams.

#### Inconsistency in the style of column names needed changing before further analysis 

#### Creating a new table with adjusted column names 

```SQL

CREATE TABLE karasdata.kjams AS
SELECT
    endTime AS endtime,
    artistName AS artistname,
    trackName AS trackname,
    msPlayed AS msplayed,
    secsplayed,
    minsplayed,
    day_of_week AS dayofweek,
    time_of_day AS timeofday
FROM
    karasdata.jams

```

#### Dropping old table 

```SQL
DROP TABLE karasdata.jam;
```


---
### Exploration

#### Top 10 artists by playtime 

```SQL
SELECT 
  artistname, 
  SUM(minsplayed) AS minsplayedtotal
FROM 
  `bamboo-life-418613.karasdata.karasjams`
GROUP BY 
  artistname
ORDER BY 
  minsplayedtotal DESC
LIMIT 10;
```
| artistname           | minsplayedtotal |
|----------------------|----------------:|
| Orion Sun            | 2,495.31        |
| Remi Wolf            | 1,358.47        |
| SZA                  | 1,355.90        |
| Mt. Joy              | 1,285.24        |
| Doja Cat             | 1,215.60        |
| Shakey Graves        | 831.35          |
| Tyler, The Creator   | 803.36          |
| Billie Eilish        | 719.83          |
| The Backseat Lovers  | 627.24          |
| Still Woozy          | 499.46          |

```SQL
SELECT 
  Hours_Listening,
  (Hours_Listening / 24) AS Days_Listening
FROM (
  SELECT 
    SUM(minsplayed) / 60 AS Hours_Listening
  FROM 
    karasdata.kjams
) AS Subquery;
```

| Hours_Listening        | Days_Listening        |
|------------------------|-----------------------|
| 940.20535916666051     | 39.175223298610852    |


#### Top 10 songs by listening time 
``` SQL
SELECT 
  trackname, 
  artistname,
  SUM(minsplayed) AS totalplaytime
FROM 
  `karasdata.kjams`
GROUP BY 
  trackname,
  artistname
ORDER BY 
  totalplaytime DESC
LIMIT 
  10;
```

# Top 10 Tracks by Total Play Time

| trackname                               | artistname            | totalplaytime |
|-----------------------------------------|-----------------------|---------------|
| concrete                                | Orion Sun             | 373.3475      |
| dirty dancer                            | Orion Sun             | 371.3588      |
| Bathroom Light                          | Mt. Joy               | 365.6171      |
| Sweet Jane - Full Length Version; 2015 Remaster | The Velvet Underground | 307.2752      |
| Shirt                                   | SZA                   | 290.8872      |
| Space Jam - An Odyssey                  | Orion Sun             | 275.5432      |
| Cinderella                              | Remi Wolf             | 257.8302      |
| Maple Syrup                             | The Backseat Lovers   | 235.4541      |
| Kill Bill                               | SZA                   | 232.2532      |
| Clay Pigeons                            | Michael Cera          | 222.9267      |

```SQL
SELECT
  MIN(playtime) AS datastartdate,
  MAX(playtime) AS dataenddate,
  TIMESTAMP_DIFF(MAX(playtime), MIN(playtime), DAY) AS days_difference,
  TIMESTAMP_DIFF(MAX(playtime), MIN(playtime), MINUTE) AS minutes_difference
FROM 
  `karasdata.kjams`;
```



### Exploring what % of my time was spent listening to music.

To do this the following steps were taken 
1) Find start time and end time of the dataset 
2) Calculation of minutes between these two datetimes
3) Use sum all records playtime to find the percent of step 2s total

```SQL
SELECT
  MIN(playtime) AS datastartdate,
  MAX(playtime) AS dataenddate,
  TIMESTAMP_DIFF(MAX(playtime), MIN(playtime), DAY) AS days_difference,
  TIMESTAMP_DIFF(MAX(playtime), MIN(playtime), MINUTE) AS minutes_difference
FROM 
  `karasdata.kjams`;
```


| datastartdate          | dataenddate            | days_difference | minutes_difference |
|------------------------|------------------------|----------------:|-------------------:|
| 2023-05-31 20:28:00 UTC | 2024-06-01 23:57:00 UTC | 367             | 528,689           |

#### Cartesian Product / Cross Join had to be used for this calculation
``` SQL
WITH time_frame AS (
  SELECT
    TIMESTAMP_DIFF(MAX(endtime), MIN(endtime), MINUTE) AS total_minutes
  FROM 
    karasdata.kjams
),
listening_time AS (
  SELECT
    SUM(msplayed) / 60000 AS total_listening_minutes
  FROM 
    karasdata.kjams
)
SELECT
  total_listening_minutes,
  total_minutes,
  (total_listening_minutes / total_minutes) * 100 AS percent_listening_time
FROM
  time_frame, listening_time;

```


This table shows the total listening minutes, total minutes, and the percentage of time spent listening to music based on the analysis of the dataset.

| total_listening_minutes | total_minutes | percent_listening_time |
|-------------------------|---------------|------------------------|
| 56412.32155             | 528689        | 10.670227969562445     |


- **Total Listening Minutes**: The total amount of time spent listening to music in minutes.
- **Total Minutes**: The total duration from the start to the end of the dataset in minutes.
- **Percent Listening Time**: The percentage of total time that was spent listening to music.


____


As I moved into the next phase of my analysis, I became curious about the specific listening patterns of each of my top artists.
I wondered how each song in their discography contributed to my total listening per artist. 


what percent of total time (from start date to end date of data) was spent listening to music 
#### The CTES, joins and calculation get the percentage of each song's listening time relative to the total listening time for each artist, excluding percentage values lower than 1%.
#### There were too many results with an original limit of 10 artists; so I reduced the analysis to my top 3 artists (Orion Sun, Remi Wolf, SZA).
#### Results were exported to a google sheet for further analysis 
```SQL
WITH top_artists AS (
  SELECT
    artistName,
    SUM(minsplayed) AS total_artist_time
  FROM `karasdata.kjams`
  GROUP BY artistName
  ORDER BY total_artist_time DESC
  LIMIT 3
),
artist_song_time AS (
  SELECT
    artistName,
    trackName,
    SUM(minsplayed) AS total_song_time
  FROM `karasdata.kjams`
  WHERE artistname IN (SELECT artistName FROM top_artists)
  GROUP BY artistname, trackName
),
artist_total_time AS (
  SELECT
    artistName,
    SUM(total_song_time) AS total_artist_time
  FROM artist_song_time
  GROUP BY artistName
)

SELECT
  a.artistName,
  a.trackName,
  a.total_song_time,
  b.total_artist_time,
  ROUND((a.total_song_time / b.total_artist_time) * 100, 3) AS song_time_percentage
FROM artist_song_time a
JOIN artist_total_time b
ON a.artistName = b.artistName
ORDER BY a.artistName, song_time_percentage DESC;
```
---

The results of this query were exported to google sheets for further manipulation, exploration and visualizations. 



#### - I am now breaking these results into 4 different sheets, one for a compilation of the top 3 artists data together and one for each artist with their data separated. 

![*sheets preview.png*](https://github.com/karammulc/Karas-Jams/blob/main/Images/SheetsPreview.png)

Using visualization tools within sheets I then created donut charts for each artists discography, this can be seen within the README. 


Moving into Looker Studio was a new experience for me! So, I began playing around with different options.
The following was created 

- A total song count (calculated by sum(recordcount)


When creating the barchart for listening duration by day of week; the options were to order by Dimension (dayofweek) - alphabetically, or Metric (msplayed)
However neither of these options ordered the X axis as Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday

To remedy this I added a new calculated field using this code:
```
CASE
  WHEN dayofweek = 'Monday' THEN 1
  WHEN dayofweek = 'Tuesday' THEN 2
  WHEN dayofweek = 'Wednesday' THEN 3
  WHEN dayofweek = 'Thursday' THEN 4
  WHEN dayofweek = 'Friday' THEN 5
  WHEN dayofweek = 'Saturday' THEN 6
  WHEN dayofweek = 'Sunday' THEN 7
END
```

This offered the opportunity to order the X axis as I wish.

It bothered me that for the donut chart representing listening by time of day , morning,afternoon and night weren't capitalized so I added one more calculated field.

```
CASE
  WHEN timeofday = 'morning' THEN 'Morning'
  WHEN timeofday = 'afternoon' THEN 'Afternoon'
  WHEN timeofday = 'night' THEN 'Night'END

```


# Key Findings
______________________________________________________
#### Total Counts of songs listened to: *37,527*
______________________________________________________
#### Percent of Total Time Listening to Music: 	*10.67%*
______________________________________________________
#### Hours Listening: 940  /  Days Listening: *39* 
______________________________________________________

### Top 10 artists 
1. Orion Sun
2. Remi Wolf
3. SZA
4. Mt. Joy
5. Doja Cat
6. Shakey Graves
7. Tyler, The Creator
8. Billie Eilish 
9. The Backseat Lovers
10. Still Woozy
_____________________________________________________
### Top 10 songs overall
1. Concrete - Orion Sun
2. Dirty dancer - Orion Sun
3. Bathroom Light - Mnt. Joy
4. Sweet Jane - The Velvet Underground
5. Shirt - SZA
6. Space Jam - An Odyssey	- Orion Sun
7. Cinderella - Remi Wolf
8. Maple Syrup - The BackSeat Lovers
9. Kill Bill - SZA
10. Clay Pigeons- Michael Cera	
_____________________________________________________
### Top 5 songs for my top 3 artists 

#### Orion Sun
1. Concrete - 15%
2. Dirty Dancer - 14.9%
3. Space Jam an Odyssey - 11%
4. Antidote - 5.6%
5. Lightning - 4.5%
______________________________________________________
![*Song Time Percentage For Orion Sun Tracks*](https://github.com/karammulc/Karas-Jams/blob/main/Images/Song%20Time%20Percentage%20For%20Orion%20Sun%20Tracks.png)
______________________________________________________
#### Remi Wolf 
1. Cinderella - 19%
2. Toro - 8.4%
3. Shawty - 8.1%
4. Sugar - 7.4%
5. Photo Id - 7.0 %
______________________________________________________
![*Song Time Percentage For Remi Wolf Tracks*](https://github.com/karammulc/Karas-Jams/blob/main/Images/Song%20Time%20Percentage%20For%20Remi%20Wolf%20Tracks.png)
______________________________________________________
#### SZA
1. Shirt - 21.5%
2. Kill Bill - 17.1%
3. Normal Girl - 11.9%
4. Saturn Instrumental - 5.6%
5. Saturn - 4.4%
______________________________________________________
![*Song Time Percentage For SZA Tracks*](https://github.com/karammulc/Karas-Jams/blob/main/Images/Song%20Time%20Percentage%20For%20SZA%20Tracks.png)
______________________________________________________
### Listening by Time of Day
Morning: 16%
Afternoon: 50.5%
Night: 33.6%
______________________________________________________
### Listening by Day of Week

Monday - 11.7%

Tuesday - 15.2%

Wednesday - 12.6%

Thursday - 15.1%

Friday - 17.2%

Saturday - 14.6%

Sunday - 13.6%
______________________________________________________

