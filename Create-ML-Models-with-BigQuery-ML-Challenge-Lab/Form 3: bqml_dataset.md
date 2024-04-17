If you find my repository helpful, please star⭐ it 🌟.

## Task-1: Create a new dataset with dataset id 'bq_dataset'
1. In the Explorer pane of the BigQuery Console, click on the three dots next to your Project ID and then select Create dataset.
2. In the Create dataset dialog, set the Dataset ID to `bq_dataset` and leave the other options at their default values.
3. Click __Create dataset__.

OR
      
1. Open Cloud Shell
2. Click Authorize
```
bq mk bq_dataset
```
4. Dataset will be created

##  Task-2: Create and evaluate a model
You have precreated dataset `bqml_dataset`. You have to create a model that predicts whether a visitor will make a transaction.
Also, evaluate the newely created model.
 
1. Open the Explorer pane of the BigQuery Console,
2. Click "+" (Compose New Query) icon and then run the following query:
 
 
Creation of a model
```
#standardSQL

CREATE OR REPLACE MODEL `bqml_dataset.predicts_visitor_model`

OPTIONS(model_type='logistic_reg') AS

SELECT

IF(totals.transactions IS NULL, 0, 1) AS label,

IFNULL(device.operatingSystem, "") AS os,

device.isMobile AS is_mobile,

IFNULL(geoNetwork.country, "") AS country,

IFNULL(totals.pageviews, 0) AS pageviews

FROM

`bigquery-public-data.google_analytics_sample.ga_sessions_*`

WHERE

_TABLE_SUFFIX BETWEEN '20160801' AND '20170631'

LIMIT 100000;

 
 
Evaluation of a model
 
#standardSQL

SELECT

*

FROM

ml.EVALUATE(MODEL `bqml_dataset.predicts_visitor_model`, (

SELECT

IF(totals.transactions IS NULL, 0, 1) AS label,

IFNULL(device.operatingSystem, "") AS os,

device.isMobile AS is_mobile,

IFNULL(geoNetwork.country, "") AS country,

IFNULL(totals.pageviews, 0) AS pageviews

FROM

`bigquery-public-data.google_analytics_sample.ga_sessions_*`

WHERE

_TABLE_SUFFIX BETWEEN '20170701' AND '20170801'));
```

## Task-3:Use the model to predict purchases per country
You have created model `predict_purchase_model` in the dataset `bqml_dataset` to predict purchases per country.
Run the query to predict the number of transactions made by visitors of each country, sort the results, and select the top 10 countries by purchases.
 
1. Open the Explorer pane of the BigQuery Console,
2. Click "+" (Compose New Query) icon and then run the following query:

```
#standardSQL

SELECT

country,

SUM(predicted_label) as total_predicted_purchases

FROM

ml.PREDICT(MODEL `bqml_dataset.predict_purchase_model`, (

SELECT

IFNULL(device.operatingSystem, "") AS os,

device.isMobile AS is_mobile,

IFNULL(totals.pageviews, 0) AS pageviews,

IFNULL(geoNetwork.country, "") AS country

FROM

`bigquery-public-data.google_analytics_sample.ga_sessions_*`

WHERE

_TABLE_SUFFIX BETWEEN '20170701' AND '20170801'))

GROUP BY country

ORDER BY total_predicted_purchases DESC

LIMIT 10;
```

## Task-4: Use the model to predict purchases per user
You have created model `predict_purchase_model` in the dataset `bqml_dataset` to predict purchases per country.
Run the query to predict the number of transactions each visitor makes, sort the results, and select the top 10 visitors by transactions.

1. Open the Explorer pane of the BigQuery Console,
2. Click "+" (Compose New Query) icon and then run the following query:
 
```
#standardSQL

SELECT

fullVisitorId,

SUM(predicted_label) as total_predicted_purchases

FROM

ml.PREDICT(MODEL `bqml_dataset.predict_purchase_model`, (

SELECT

IFNULL(device.operatingSystem, "") AS os,

device.isMobile AS is_mobile,

IFNULL(totals.pageviews, 0) AS pageviews,

IFNULL(geoNetwork.country, "") AS country,

fullVisitorId

FROM

`bigquery-public-data.google_analytics_sample.ga_sessions_*`

WHERE

_TABLE_SUFFIX BETWEEN '20170701' AND '20170801'))

GROUP BY fullVisitorId

ORDER BY total_predicted_purchases DESC

LIMIT 10;
``` 
