
The data used for this project was requested directly from Spotify within personal account settings.
Four separate json files were received but were combined prior to upload by utilizing Python. 
This process resulted in one large CSV. 

I am new to Python, so Claude AI was leveraged for assistance in this process.

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

The first step in manipulation is to add various time-based columns. 
The addition of secplayed and minplayed is for easier readability than milliseconds.
Time of day and day of week are to be added for a more extensive temporal analysis. 
In updating all rows for new columns; a WHERE TRUE statement is used as bigquery requires WHERE statements while using UPDATE.


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


Next, I was interested in finding what percent of total time (from start date to end date of data) was spent listening to music 
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

#### Cartesian Product / Cross Join must be used for this calculation
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



#### The CTES, joins and calculation get the percentage of each song's listening time relative to the total listening time for each artist, excluding percentage values lower than 1%.
#### There were too many results with an original limit of 10 artists; so I reduced the analysis to my top 3 artists (Orion Sun, Remi Wolf, SZA).
#### Results were exported to a google sheet for further analysis 
```SQL
WITH top_artists AS (
  SELECT
    artistName,
    SUM(msPlayed) AS total_artist_time
  FROM `karasdata.kjams`
  GROUP BY artistName
  ORDER BY total_artist_time DESC
  LIMIT 3
),
artist_song_time AS (
  SELECT
    artistName,
    trackName,
    SUM(msPlayed) AS total_song_time
  FROM `karasdata.kjams`
  WHERE artistName IN (SELECT artistName FROM top_artists)
  GROUP BY artistName, trackName
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
  ROUND((a.total_song_time / b.total_artist_time) * 100, 2) AS song_time_percentage
FROM artist_song_time a
JOIN artist_total_time b
ON a.artistName = b.artistName
WHERE ROUND((a.total_song_time / b.total_artist_time) * 100, 2) > 1
ORDER BY a.artistName, song_time_percentage DESC;
```
---

#### - I am now breaking these results into 3 different sheets, one for each artist
#### - Before further analysis some manipulating will be done within sheets 
#### - This query used ms so within my google sheet I will be making a column converting ms to hours

##1 hour = 3,600,000 milliseconds


