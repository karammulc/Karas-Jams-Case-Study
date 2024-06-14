
#### Adding secplayed column 
```SQL
ALTER TABLE karasdata.jams
ADD COLUMN secsplayed FLOAT64;
```
#### Update secplayed column 
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
## I made a backup 

```SQL
DELETE FROM karasjams.jams
WHERE msplayed = 0;
```

Upon exploration, I found that some records had 0 ms played but the records still existed.
In my cleaning removed these records.

####This statement removed 962 rows of 38489 from jams.
