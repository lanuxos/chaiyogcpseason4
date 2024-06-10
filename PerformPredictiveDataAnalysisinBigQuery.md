# Perform Predictive Data Analysis in BigQuery
## BigQuery Soccer Data Ingestion
### Open BigQuery
BigQuery
Done

### Create custom tables
Explorer
three dots
Create dataset

Dataset ID	                soccer
Data location	            us (multiple regions in United States)
Default table expiration	Default

Create dataset

### Load JSON Data
View actions [three dots]
Create table

Create table from	        Google Cloud Storage
Select file from GCS bucket	spls/bq-soccer-analytics/competitions.json
File format	                JSONL (Newline delimited JSON)
Table name	                competitions
Schema	                    Check the box marked Schema Auto detect

Create table

### Load CSV data
View actions [three dots]
Create table

Create table from	        Google Cloud Storage
Select file from GCS bucket	spls/bq-soccer-analytics/tags2name.csv
File format	                CSV
Table name	                tags2name
Schema	                    Check the box marked Auto detect

Create table

matches
spls/bq-soccer-analytics/matches.json

teams
spls/bq-soccer-analytics/teams.json

players
spls/bq-soccer-analytics/players.json

events
spls/bq-soccer-analytics/events.json

### Preview tables
soccer
competitions
Preview

### Query Player data
- create sql query
```
SELECT
  (firstName || ' ' || lastName) AS player,
  birthArea.name AS birthArea,
  height
FROM
  `soccer.players`
WHERE
  role.name = 'Defender'
ORDER BY
  height DESC
LIMIT 5
```

### Query events data
- Create a query to retrieve counts of all event types that are found in the events table

```
SELECT
  eventId,
  eventName,
  COUNT(id) AS numEvents
FROM
  `soccer.events`
GROUP BY
  eventId, eventName
ORDER BY
  numEvents DESC
```
### Pop quiz

## BigQuery Soccer Data Analysis [GSP849]
### Open BigQuery
BigQuery
Done

### Matches with the most goals
- analytics such as the most total goals scored in a match by both teams
```
SELECT
 date,
 label,
 (team1.score + team2.score) AS totalGoals
FROM
 `soccer.matches` Matches
LEFT JOIN
 `soccer.competitions` Competitions ON
   Matches.competitionId = Competitions.wyId
WHERE
 status = 'Played' AND
 Competitions.name = 'Spanish first division'
ORDER BY
 totalGoals DESC, date DESC
```

### Players with the most passes
- analytics such as total passes by players
```
SELECT
 playerId,
 (Players.firstName || ' ' || Players.lastName) AS playerName,
 COUNT(id) AS numPasses

FROM
 `soccer.events` Events

LEFT JOIN
 `soccer.players` Players ON
   Events.playerId = Players.wyId

WHERE
 eventName = 'Pass'

GROUP BY
 playerId, playerName

ORDER BY
 numPasses DESC

LIMIT 10
```

### Determine penalty kick success rate
- analytics such as the success rate on penalty kicks by each player
```
SELECT
 playerId,
 (Players.firstName || ' ' || Players.lastName) AS playerName,
 COUNT(id) AS numPKAtt,
 SUM(IF(101 IN UNNEST(tags.id), 1, 0)) AS numPKGoals,

 SAFE_DIVIDE(
   SUM(IF(101 IN UNNEST(tags.id), 1, 0)),
   COUNT(id)
   ) AS PKSuccessRate

FROM
 `soccer.events` Events

LEFT JOIN
 `soccer.players` Players ON
   Events.playerId = Players.wyId

WHERE
 eventName = 'Free Kick' AND
 subEventName = 'Penalty'

GROUP BY
 playerId, playerName

HAVING
 numPkAtt >= 5

ORDER BY
 PKSuccessRate DESC, numPKAtt DESC
```

### Pop quiz


## BigQuery Soccer Data Analytical Insigt
### Open BigQuery
- 

### Analyze nested soccer event data
- JOINs with BigQuery's array functionality to enable better control over the soccer event data
```
SELECT
 Events.playerId,
 (Players.firstName || ' ' || Players.lastName) AS playerName,
 SUM(IF(Tags2Name.Label = 'assist', 1, 0)) AS numAssists

FROM
 `soccer.events` Events,
 Events.tags Tags

LEFT JOIN
 `soccer.tags2name` Tags2Name ON
   Tags.id = Tags2Name.Tag

LEFT JOIN
 `soccer.players` Players ON
   Events.playerId = Players.wyId

GROUP BY
 playerId, playerName

ORDER BY
 numAssists DESC
```
### Calculate the average pass distance by team
- How much do club teams differ in terms of average distance on their passes (both overall and accurate ones)?
```
WITH
Passes AS
(
 SELECT
   *,
   /* 1801 is known Tag for 'accurate' from tags2name table */
   (1801 IN UNNEST(tags.id)) AS accuratePass,

   (CASE
     WHEN ARRAY_LENGTH(positions) != 2 THEN NULL
     ELSE
  /* Translate 0-100 (x,y) coordinate-based distances to absolute positions
  using "average" field dimensions of 105x68 before combining in 2D dist calc */
       SQRT(
         POW(
           (positions[ORDINAL(2)].x - positions[ORDINAL(1)].x) * 105/100,
           2) +
         POW(
           (positions[ORDINAL(2)].y - positions[ORDINAL(1)].y) * 68/100,
           2)
         )
     END) AS passDistance

 FROM
   `soccer.events`

 WHERE
   eventName = 'Pass'
)

SELECT
 Passes.teamId,
 Teams.name AS team,
 Teams.area.name AS teamArea,

 COUNT(Passes.Id) AS numPasses,
 AVG(Passes.passDistance) AS avgPassDistance,
 SAFE_DIVIDE(
   SUM(IF(Passes.accuratePass, Passes.passDistance, 0)),
   SUM(IF(Passes.accuratePass, 1, 0))
   ) AS avgAccuratePassDistance

FROM
 Passes

LEFT JOIN
 `soccer.teams` Teams ON
   Passes.teamId = Teams.wyId

WHERE
 Teams.type = 'club'

GROUP BY
 teamId, team, teamArea

ORDER BY
 avgPassDistance
```
### Analyze shot distance
- What impact does the distance of a shot have on the likelihood of a goal being scored?
```
WITH
Shots AS
(
 SELECT
  *,

  /* 101 is known Tag for 'goals' from goals table */
  (101 IN UNNEST(tags.id)) AS isGoal,

  /* Translate 0-100 (x,y) coordinate-based distances to absolute positions
  using "average" field dimensions of 105x68 before combining in 2D dist calc */
  SQRT(
    POW(
      (100 - positions[ORDINAL(1)].x) * 105/100,
      2) +
    POW(
      (50 - positions[ORDINAL(1)].y) * 68/100,
      2)
     ) AS shotDistance

 FROM
  `soccer.events`

 WHERE
  /* Includes both "open play" & free kick shots (including penalties) */
  eventName = 'Shot' OR
  (eventName = 'Free Kick' AND subEventName IN ('Free kick shot', 'Penalty'))
)

SELECT
 ROUND(shotDistance, 0) AS ShotDistRound0,

 COUNT(*) AS numShots,
 SUM(IF(isGoal, 1, 0)) AS numGoals,
 AVG(IF(isGoal, 1, 0)) AS goalPct

FROM
 Shots

WHERE
 shotDistance <= 50

GROUP BY
 ShotDistRound0

ORDER BY
 ShotDistRound0
```
### Analyze shot angle
```
WITH
Shots AS
(
 SELECT
  *,

  /* 101 is known Tag for 'goals' from goals table */
  (101 IN UNNEST(tags.id)) AS isGoal,

  /* Translate 0-100 (x,y) coordinates to absolute positions using "average"
  field dimensions of 105x68 before using in various distance calcs;
  LEAST used to cap shot locations to on-field (x, y) (i.e. no exact 100s) */
  LEAST(positions[ORDINAL(1)].x, 99.99999) * 105/100 AS shotXAbs,
  LEAST(positions[ORDINAL(1)].y, 99.99999) * 68/100 AS shotYAbs

 FROM
   `soccer.events`

 WHERE
   /* Includes both "open play" & free kick shots (including penalties) */
   eventName = 'Shot' OR
   (eventName = 'Free Kick' AND subEventName IN ('Free kick shot', 'Penalty'))
),

ShotsWithAngle AS
(
 SELECT
   Shots.*,

   /* Law of cosines to get 'open' angle from shot location to goal, given
    that goal opening is 7.32m, placed midway up at field end of (105, 34) */
   SAFE.ACOS(
     SAFE_DIVIDE(
       ( /* Squared distance between shot and 1 post, in meters */
         (POW(105 - shotXAbs, 2) + POW(34 + (7.32/2) - shotYAbs, 2)) +
         /* Squared distance between shot and other post, in meters */
         (POW(105 - shotXAbs, 2) + POW(34 - (7.32/2) - shotYAbs, 2)) -
         /* Squared length of goal opening, in meters */
         POW(7.32, 2)
       ),
       (2 *
         /* Distance between shot and 1 post, in meters */
         SQRT(POW(105 - shotXAbs, 2) + POW(34 + 7.32/2 - shotYAbs, 2)) *
         /* Distance between shot and other post, in meters */
         SQRT(POW(105 - shotXAbs, 2) + POW(34 - 7.32/2 - shotYAbs, 2))
       )
     )
   /* Translate radians to degrees */
   ) * 180 / ACOS(-1)
   AS shotAngle

 FROM
   Shots
)

SELECT
 ROUND(shotAngle, 0) AS ShotAngleRound0,

 COUNT(*) AS numShots,
 SUM(IF(isGoal, 1, 0)) AS numGoals,
 AVG(IF(isGoal, 1, 0)) AS goalPct

FROM
 ShotsWithAngle

GROUP BY
 ShotAngleRound0

ORDER BY
 ShotAngleRound0
```
### Pop quiz


## BigQuery Machine Learning using Soccer Data [GSP851]
### Open BigQuery
- 

### Calculate shot distance and shot angle
```
CREATE FUNCTION `soccer.GetShotDistanceToGoal`(x INT64, y INT64)
RETURNS FLOAT64
AS (
 /* Translate 0-100 (x,y) coordinate-based distances to absolute positions
 using "average" field dimensions of 105x68 before combining in 2D dist calc */
 SQRT(
   POW((100 - x) * 105/100, 2) +
   POW((50 - y) * 68/100, 2)
   )
 );
```
```
CREATE FUNCTION `soccer.GetShotAngleToGoal`(x INT64, y INT64)
RETURNS FLOAT64
AS (
 SAFE.ACOS(
   /* Have to translate 0-100 (x,y) coordinates to absolute positions using
   "average" field dimensions of 105x68 before using in various distance calcs */
   SAFE_DIVIDE(
     ( /* Squared distance between shot and 1 post, in meters */
       (POW(105 - (x * 105/100), 2) + POW(34 + (7.32/2) - (y * 68/100), 2)) +
       /* Squared distance between shot and other post, in meters */
       (POW(105 - (x * 105/100), 2) + POW(34 - (7.32/2) - (y * 68/100), 2)) -
       /* Squared length of goal opening, in meters */
       POW(7.32, 2)
     ),
     (2 *
       /* Distance between shot and 1 post, in meters */
       SQRT(POW(105 - (x * 105/100), 2) + POW(34 + 7.32/2 - (y * 68/100), 2)) *
       /* Distance between shot and other post, in meters */
       SQRT(POW(105 - (x * 105/100), 2) + POW(34 - 7.32/2 - (y * 68/100), 2))
     )
    )
  /* Translate radians to degrees */
  ) * 180 / ACOS(-1)
 )
;
```
### Create expected goals models using BigQuery ML
```
CREATE MODEL `soccer.xg_logistic_reg_model`
OPTIONS(
 model_type = 'LOGISTIC_REG',
 input_label_cols = ['isGoal']
 ) AS

SELECT
 Events.subEventName AS shotType,

 /* 101 is known Tag for 'goals' from goals table */
 (101 IN UNNEST(Events.tags.id)) AS isGoal,

  `soccer.GetShotDistanceToGoal`(Events.positions[ORDINAL(1)].x,
   Events.positions[ORDINAL(1)].y) AS shotDistance,

 `soccer.GetShotAngleToGoal`(Events.positions[ORDINAL(1)].x,
   Events.positions[ORDINAL(1)].y) AS shotAngle

FROM
 `soccer.events` Events

LEFT JOIN
 `soccer.matches` Matches ON
   Events.matchId = Matches.wyId

LEFT JOIN
 `soccer.competitions` Competitions ON
   Matches.competitionId = Competitions.wyId

WHERE
 /* Filter out World Cup matches for model fitting purposes */
 Competitions.name != 'World Cup' AND
 /* Includes both "open play" & free kick shots (including penalties) */
 (
   eventName = 'Shot' OR
   (eventName = 'Free Kick' AND subEventName IN ('Free kick shot', 'Penalty'))
 )
;
```
- Create a boosted tree model for expected goals
```
CREATE MODEL `soccer.xg_boosted_tree_model`
OPTIONS(
 model_type = 'BOOSTED_TREE_CLASSIFIER',
 input_label_cols = ['isGoal']
 ) AS

SELECT
 Events.subEventName AS shotType,

 /* 101 is known Tag for 'goals' from goals table */
 (101 IN UNNEST(Events.tags.id)) AS isGoal,
  `soccer.GetShotDistanceToGoal`(Events.positions[ORDINAL(1)].x,
   Events.positions[ORDINAL(1)].y) AS shotDistance,

 `soccer.GetShotAngleToGoal`(Events.positions[ORDINAL(1)].x,
   Events.positions[ORDINAL(1)].y) AS shotAngle

FROM
 `soccer.events` Events

LEFT JOIN
 `soccer.matches` Matches ON
   Events.matchId = Matches.wyId

LEFT JOIN
 `soccer.competitions` Competitions ON
   Matches.competitionId = Competitions.wyId

WHERE
 /* Filter out World Cup matches for model fitting purposes */
 Competitions.name != 'World Cup' AND
 /* Includes both "open play" & free kick shots (including penalties) */
 (
   eventName = 'Shot' OR
   (eventName = 'Free Kick' AND subEventName IN ('Free kick shot', 'Penalty'))
 )
;
```

### Apply an expected goals model to new data
- Get probabilities for all shots in 2018 World Cup
```
SELECT
 *

FROM
 ML.PREDICT(
   MODEL `soccer.xg_logistic_reg_model`,
   (
     SELECT
       Events.subEventName AS shotType,

       /* 101 is known Tag for 'goals' from goals table */
       (101 IN UNNEST(Events.tags.id)) AS isGoal,

       `soccer.GetShotDistanceToGoal`(Events.positions[ORDINAL(1)].x,
           Events.positions[ORDINAL(1)].y) AS shotDistance,

       `soccer.GetShotAngleToGoal`(Events.positions[ORDINAL(1)].x,
           Events.positions[ORDINAL(1)].y) AS shotAngle

     FROM
       `soccer.events` Events

     LEFT JOIN
       `soccer.matches` Matches ON
           Events.matchId = Matches.wyId

     LEFT JOIN
       `soccer.competitions` Competitions ON
           Matches.competitionId = Competitions.wyId

     WHERE
       /* Look only at World Cup matches for model predictions */
       Competitions.name = 'World Cup' AND
       /* Includes both "open play" & free kick shots (including penalties) */
       (
           eventName = 'Shot' OR
           (eventName = 'Free Kick' AND subEventName IN ('Free kick shot', 'Penalty'))
       )
   )
 )
```
- Identify most unlikely goals using model probabilities
```
SELECT
 predicted_isGoal_probs[ORDINAL(1)].prob AS predictedGoalProb,
 * EXCEPT (predicted_isGoal, predicted_isGoal_probs),

FROM
 ML.PREDICT(
   MODEL `soccer.xg_logistic_reg_model`,
   (
     SELECT
       Events.playerId,
       (Players.firstName || ' ' || Players.lastName) AS playerName,

       Teams.name AS teamName,
       CAST(Matches.dateutc AS DATE) AS matchDate,
       Matches.label AS match,

     /* Convert match period and event seconds to minute of match */
       CAST((CASE
         WHEN Events.matchPeriod = '1H' THEN 0
         WHEN Events.matchPeriod = '2H' THEN 45
         WHEN Events.matchPeriod = 'E1' THEN 90
         WHEN Events.matchPeriod = 'E2' THEN 105
         ELSE 120
         END) +
         CEILING(Events.eventSec / 60) AS INT64)
         AS matchMinute,

       Events.subEventName AS shotType,

       /* 101 is known Tag for 'goals' from goals table */
       (101 IN UNNEST(Events.tags.id)) AS isGoal,

       `soccer.GetShotDistanceToGoal`(Events.positions[ORDINAL(1)].x,
           Events.positions[ORDINAL(1)].y) AS shotDistance,

       `soccer.GetShotAngleToGoal`(Events.positions[ORDINAL(1)].x,
           Events.positions[ORDINAL(1)].y) AS shotAngle

     FROM
       `soccer.events` Events

     LEFT JOIN
       `soccer.matches` Matches ON
           Events.matchId = Matches.wyId

     LEFT JOIN
       `soccer.competitions` Competitions ON
           Matches.competitionId = Competitions.wyId

     LEFT JOIN
       `soccer.players` Players ON
           Events.playerId = Players.wyId

     LEFT JOIN
       `soccer.teams` Teams ON
           Events.teamId = Teams.wyId

     WHERE
       /* Look only at World Cup matches to apply model */
       Competitions.name = 'World Cup' AND
       /* Includes both "open play" & free kick shots (but not penalties) */
       (
         eventName = 'Shot' OR
         (eventName = 'Free Kick' AND subEventName IN ('Free kick shot'))
       ) AND
       /* Filter only to goals scored */
       (101 IN UNNEST(Events.tags.id))
   )
 )

ORDER BY
  predictedgoalProb
```
### Pop quiz
- 


## Perform Predictive Data Analysis in BigQuery: Challenge Lab [GSP374]
Challenge scenario
Use BigQuery to load the data from the Cloud Storage bucket, write and execute queries in BigQuery, analyze soccer event data. Then use BigQuery ML to train an expected goals model on the soccer event data and evaluate the impressiveness of World Cup goals.
### Data Ingestion
- load json
Source	                                Cloud Storage
Select file from Cloud Storage bucket	spls/bq-soccer-analytics/events.json
File format	                            JSONL (Newline delimited JSON)
Table name	                            Table name
Schema	                                Check the box marked Schema Auto detect
- load csv
Source	                                Cloud Storage
Select file from Cloud Storage bucket	spls/bq-soccer-analytics/tags2name.csv
File format	                            CSV
Table name	                            Table name
Schema	                                Check the box marked Auto detect
### Analyze soccer data
- 

### Gain insight by analyzing soccer data
- 

### Create a regression model using soccer data
- 

### Make predictions from new data with the BigQuery model
- 

