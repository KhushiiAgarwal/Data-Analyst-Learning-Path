If you find my repository helpful, please star⭐ it 🌟.

## Task-1: Create a new dataset with dataset id 'bq_dataset'
1. In the Explorer pane of the BigQuery Console, click on the three dots next to your Project ID and then select Create dataset.
2. In the Create dataset dialog, set the Dataset ID to `bq_dataset` and leave the other options at their default values.
3. Click __Create dataset__.

OR
      
1. Open Cloud Shell
2. Click Authorize
3. ``` bq mk bq_dataset```
4. Dataset will be created

## Task-2: Create a forecasting BigQuery machine learning model
You have already created `austin` dataset. In this task you have to create the machine learning model named `austin_location_model` to predict the trip duration for bike trips.
 
1. Open the Explorer pane of the BigQuery Console,
2. Click "+" (Compose New Query) icon and then run the following query:
   ```
      CREATE OR REPLACE MODEL austin.austin_location_model
    
    OPTIONS
    
    (model_type='linear_reg', labels=['duration_minutes']) AS
    
    SELECT
    
      start_station_name,
    
      EXTRACT(HOUR FROM start_time) AS start_hour,
    
      EXTRACT(DAYOFWEEK FROM start_time) AS day_of_week,
    
      duration_minutes,
    
      address as location
    
    FROM
    
      `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS trips
    
    JOIN
    
      `bigquery-public-data.austin_bikeshare.bikeshare_stations` AS stations
    
    ON
    
      trips.start_station_name = stations.name
    
    WHERE
    
      EXTRACT(YEAR FROM start_time) = 2019
    
      AND duration_minutes > 0
    ```
   
   ## Task-3: Evaluate the two machine learning models
 
 
You have already created machine learning models in the dataset `austin` against `2019`. You have to evaluate each of the precreated machine learning models data only using separate queries.
Your queries must report both the Mean Absolute Error and the Root Mean Square Error.
 
1. Open the Explorer pane of the BigQuery Console,
2. Click "+" (Compose New Query) icon and then run the following query:
 
Evaluation metrics for `location_model`
```
SELECT

SQRT(mean_squared_error) AS rmse,

mean_absolute_error

FROM

ML.EVALUATE(MODEL austin.location_model, (

SELECT

  start_station_name,

  EXTRACT(HOUR FROM start_time) AS start_hour,

  EXTRACT(DAYOFWEEK FROM start_time) AS day_of_week,

  duration_minutes,

  address as location

FROM

  `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS trips

JOIN

 `bigquery-public-data.austin_bikeshare.bikeshare_stations` AS stations

ON

  trips.start_station_name = stations.name

WHERE EXTRACT(YEAR FROM start_time) = 2019)

)
```
3. Evaluation metrics for `subscriber_model`
```
SELECT

SQRT(mean_squared_error) AS rmse,

mean_absolute_error

FROM

ML.EVALUATE(MODEL austin.subscriber_model, (

SELECT

  start_station_name,

  EXTRACT(HOUR FROM start_time) AS start_hour,

  subscriber_type,

  duration_minutes

FROM

  `bigquery-public-data.austin_bikeshare.bikeshare_trips` AS trips

WHERE

  EXTRACT(YEAR FROM start_time) = 2019)

)
```

 ## Task-4: Use the subscriber type machine learning model to predict average trip durations
you have models precreated and evaluated, in the dataset `austin` use the model, that uses `subscriber_type` as a feature, to predict average trip length for trips from the busiest bike sharing station in `2019` where the subscriber type is Single Trip.
 
1. Open the Explorer pane of the BigQuery Console,
2. Click "+" (Compose New Query) icon and then run the following query:
```
SELECT

start_station_name,

COUNT(*) AS trips

FROM

`bigquery-public-data.austin_bikeshare.bikeshare_trips`

WHERE

EXTRACT(YEAR FROM start_time) = 2019

GROUP BY

start_station_name

ORDER BY

trips DESC
```
3. Predict trip length.
```
SELECT AVG(predicted_duration_minutes) AS average_predicted_trip_length

FROM ML.predict(MODEL austin.subscriber_model, (

SELECT

  start_station_name,

  EXTRACT(HOUR FROM start_time) AS start_hour,

  subscriber_type,

  duration_minutes

FROM

`bigquery-public-data.austin_bikeshare.bikeshare_trips`

WHERE 

EXTRACT(YEAR FROM start_time) = 2019

AND subscriber_type = 'Single Trip'

AND start_station_name = '21st & Speedway @PCL'))
```
