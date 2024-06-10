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

## BigQuery Soccer Data Analysis
### Open BigQuery
- 

### Matches with the most goals
- 

### Players with the most passes
- 

### Determine penalty kick success rate
- 

### Pop quiz
- 


## BigQuery Soccer Data Analytical Insigt
### Open BigQuery
- 

### Analyze nested soccer event data
- 

### Calculate the average pass distance by team
- 

### Analyze shot distance
- 

### Analyze shot angle
- 

### Pop quiz
- 


## BigQuery Machine Learning using Soccer Data
### Open BigQuery
- 

### Calculate shot distance and shot angle
- 

### Create expected goals models using BigQuery ML
- 

### Apply an expected goals model to new data
- 

### Pop quiz
- 


## Perform Predictive Data Analysis in BigQuery: Challenge Lab [GSP374]
Challenge scenario
Use BigQuery to load the data from the Cloud Storage bucket, write and execute queries in BigQuery, analyze soccer event data. Then use BigQuery ML to train an expected goals model on the soccer event data and evaluate the impressiveness of World Cup goals.
### Data Ingestion
- 

### Analyze soccer data
- 

### Gain insight by analyzing soccer data
- 

### Create a regression model using soccer data
- 

### Make predictions from new data with the BigQuery model
- 

