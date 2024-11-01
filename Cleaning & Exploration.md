# Data Processing and Analysis Documentation

## Initial Data Preparation

The data used for this project was requested directly from Spotify within personal account settings. Four separate JSON files were received and combined using Python to create one consolidated CSV file.

### Python Data Consolidation
```python
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

## Data Quality Checks

### Duplicate Check
```sql
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
    trackName,
    msplayed
HAVING
    COUNT(*) > 1;
```
**Result**: No duplicates found

### Null Value Check
```sql
SELECT *
FROM `karasdata.jams`
WHERE endtime IS NULL
   OR artistName IS NULL
   OR trackName IS NULL
   OR msplayed IS NULL
   OR endTime IS NULL;
```
**Result**: No null values found

## Data Enhancement

### Adding Time-Based Columns
The following steps were taken to add various time-based columns for enhanced analysis:
- Added seconds played for better readability than milliseconds
- Added minutes played for aggregation purposes
- Added day of week for temporal analysis
- Added time of day for usage pattern analysis

Note: BigQuery requires WHERE statements while using UPDATE, hence WHERE TRUE is used.

#### Adding and Updating Seconds Played
```sql
ALTER TABLE karasdata.jams
ADD COLUMN secsplayed FLOAT64;

UPDATE karasdata.jams
SET secsplayed = msplayed / 1000
WHERE TRUE;
```

#### Adding and Updating Minutes Played
```sql
ALTER TABLE karasdata.jams
ADD COLUMN minsplayed FLOAT64;

UPDATE karasdata.jams
SET minsplayed = secsplayed / 60 
WHERE TRUE;
```

#### Adding and Updating Day of Week
```sql
ALTER TABLE karasdata.jams
ADD COLUMN day_of_week STRING;

UPDATE karasdata.jams
SET day_of_week = FORMAT_TIMESTAMP('%A', TIMESTAMP(endtime))
WHERE TRUE;
```

#### Adding and Updating Time of Day
```sql
ALTER TABLE karasdata.jams
ADD COLUMN time_of_day STRING;

UPDATE karasdata.jams
SET time_of_day = (
  CASE
    WHEN EXTRACT(HOUR FROM TIMESTAMP(endtime)) BETWEEN 9 AND 15 THEN 'morning'
    WHEN EXTRACT(HOUR FROM TIMESTAMP(endtime)) BETWEEN 16 AND 21 THEN 'afternoon'
    ELSE 'night' 
  END
);
```

### Data Cleaning
```sql
DELETE FROM karasjams.jams
WHERE msplayed = 0;
```
**Note**: This statement removed 962 rows of 38,489 from jams. Prior to deletion, a backup was exported to a Google Cloud storage bucket and saved locally.

### Column Name Standardization
```sql
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
    karasdata.jams;

DROP TABLE karasdata.jam;
```

## Data Analysis

### Top Artists Analysis
```sql
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

### Listening Time Analysis
```sql
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

### Top Tracks Analysis
```sql
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
LIMIT 10;
```

### Time Period Analysis
```sql
SELECT
  MIN(playtime) AS datastartdate,
  MAX(playtime) AS dataenddate,
  TIMESTAMP_DIFF(MAX(playtime), MIN(playtime), DAY) AS days_difference,
  TIMESTAMP_DIFF(MAX(playtime), MIN(playtime), MINUTE) AS minutes_difference
FROM 
  `karasdata.kjams`;
```

### Listening Time Percentage Calculation
```sql
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

### Artist-Specific Analysis
```sql
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

## Dashboard Development

### Calculated Fields
1. Day of Week Ordering
```sql
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

2. Time of Day Capitalization
```sql
CASE
  WHEN timeofday = 'morning' THEN 'Morning'
  WHEN timeofday = 'afternoon' THEN 'Afternoon'
  WHEN timeofday = 'night' THEN 'Night'
END
```

### Dashboard Components
- Parameters were created for date range selection and artist filtering
- Calculated fields were implemented for proper sorting and formatting
- Filters were added for interactive data exploration
- Custom visualizations were created for temporal analysis
- Record count calculations were implemented for song tracking

The results were exported to Google Sheets for additional manipulation, exploration, and visualization creation, including donut charts for artist discography analysis.
