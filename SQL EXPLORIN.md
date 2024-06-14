


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

