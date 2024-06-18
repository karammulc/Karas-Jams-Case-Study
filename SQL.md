
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
#### Checking for duplicates
```SQL
SELECT
    endtime,
    artistName,
    trackName,
    COUNT(*) AS count
FROM
    `bamboo-life-418613.karasdata.jams`
GROUP BY
    endtime,
    artistName,
    trackName
HAVING
    COUNT(*) > 1;
```

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
)
WHERE TRUE;
```


```SQL
DELETE FROM karasjams.jams
WHERE msplayed = 0;
```

Upon exploration, I found that some records had 0 ms played but the records still existed.
I removed these records during my cleaning. Prior to deletion, a backup was exported to a Google Cloud storage bucket and exported for local storage.

#### This statement removed 962 rows of 38489 from jams.

#### Inconsistency in the style of column names needed changing before further analysis 

#### Creating a new table 
```SQL

CREATE TABLE karasdata.karasjams AS
SELECT
    endTime AS endtime,
    artistName AS artistname,
    trackName AS trackName,
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
Exploration

```SQL
SELECT 
  Hours_Listening,
  (Hours_Listening / 24) AS Days_Listening
FROM (
  SELECT 
    SUM(minsplayed) / 60 AS Hours_Listening
  FROM 
    karasdata.jams
) AS Subquery;
```

| Hours_Listening        | Days_Listening        |
|------------------------|-----------------------|
| 940.20535916666051     | 39.175223298610852    |


