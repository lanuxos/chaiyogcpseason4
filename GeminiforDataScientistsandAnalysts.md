# Gemini for Data Scientists and Analysts
## Introducing Gemini for data professionals
## Analyze data with Gemini assistance
### configure your environment and account
- set environment
PROJECT_ID=$(gcloud config get-value project)
REGION=set at lab start
echo "PROJECT_ID=${PROJECT_ID}"
echo "REGION=${REGION}"
- store signed-in google user account in an environment
USER=$(gcloud config get-value account 2> /dev/null)
echo "USER=${USER}"
- enable the Cloud AI companion API for Gemini
gcloud services enable cloudaicompanion.googleapis.com --project ${PROJECT_ID}
- use Gemini grant nescessary IAM roles to lab
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member user:${USER} --role=roles/cloudaicompanion.user
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member user:${USER} --role=roles/serviceusage.serviceUsageViewer
### create a dataset and enable gemini features in bigquery
BigQuery
Explorer panel
View actions (More menu three dots icon)
Create dataset
Dataset ID	        bqml_tutorial
Location type	    select Multi-region
Create Dataset
- enable gemini features in bigquery
Auto completion
Auto generation
Explanation
### use gemini to analyze your data
Open Gemini
Start Chatting
How do I learn which datasets and tables are available to me in BigQuery?
Send prompt
Reset Chat
### prompt gemini to explain sql queries in a sales dataset
BigQuery
Compose SQL QUERY

SELECT u.id as user_id, u.first_name, u.last_name, avg(oi.sale_price) as avg_sale_price   
FROM `bigquery-public-data.thelook_ecommerce.users` as u   
JOIN `bigquery-public-data.thelook_ecommerce.order_items` as oi   
ON u.id = oi.user_id   
GROUP BY 1,2,3   
ORDER BY avg_sale_price DESC   
LIMIT 10

Explain current selection
### generate a sql query that groups sales by day and product
BigQuery
BigQuery Studio
Compose a new query
Explorer

```
# select the sum of sale_price by Date(created_at) and product_id casted to day from bigquery-public-data.thelook_ecommerce.order_id as t1 joined this with products table in the same dataset as t2
SELECT
  SUM(sale_price),
  DATE(created_at) AS created_at_day,
  CAST(product_id as INT64)
FROM
  `bigquery-public-data.thelook_ecommerce.order_items` AS t1
JOIN
  `bigquery-public-data.thelook_ecommerce.products` AS t2 ON t1.product_id = t2.id
GROUP BY
  created_at_day,
  product_id
```

### build a forecasting model and view results
- To create a forecasting ML model, in the BigQuery SQL editor
CREATE MODEL bqml_tutorial.sales_forecasting_model
OPTIONS(MODEL_TYPE='ARIMA_PLUS',
time_series_timestamp_col='date_col',
time_series_data_col='total_sales',
time_series_id_col='product_id') AS
SELECT sum(sale_price) as total_sales,
DATE(created_at) as date_col,
product_id
FROM `bigquery-public-data.thelook_ecommerce.order_items`
AS t1
INNER JOIN `bigquery-public-data.thelook_ecommerce.products`
AS t2
ON t1.product_id = t2.id
GROUP BY 2, 3;

- To get a forecast in SQL from the model
SELECT
*
FROM
  ML.FORECAST(MODEL `PROJECT_ID.DATASET_ID.MODEL_NAME`,
STRUCT(
      7 AS horizon,
      0.95 AS confidence_level
)
)

## How to analyze data with Gemini
### 
## Gemini for Data Scientists
### configure your environment and account
- set project id and region environment
PROJECT_ID=$(gcloud config get-value project)
REGION=Region
echo "PROJECT_ID=${PROJECT_ID}"
echo "REGION=${REGION}"
- store signed-in google user account in environment
USER=$(gcloud config get-value account 2> /dev/null)
echo "USER=${USER}"
- enable cloud ai companion api
gcloud services enable cloudaicompanion.googleapis.com --project ${PROJECT_ID}
- To use Gemini, grant the necessary IAM roles to your Google Cloud Qwiklabs user account
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member user:${USER} --role=roles/cloudaicompanion.user
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member user:${USER} --role=roles/serviceusage.serviceUsageViewer
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member user:${USER} --role=roles/aiplatform.notebookRuntimeUser
gcloud projects add-iam-policy-binding ${PROJECT_ID} --member user:${USER} --role=roles/dataform.codeEditor
- Add the Notebook Runtime User, Dataform Code Editor and Compute Admin roles to the Qwiklabs user service account
CLOUD_BUILD_SERVICE_ACCOUNT="${PROJECT_ID}@${PROJECT_ID}.iam.gserviceaccount.com"
gcloud projects add-iam-policy-binding $PROJECT_ID --member serviceAccount:$CLOUD_BUILD_SERVICE_ACCOUNT --role roles/aiplatform.notebookRuntimeUser
gcloud projects add-iam-policy-binding $PROJECT_ID --member serviceAccount:$CLOUD_BUILD_SERVICE_ACCOUNT --role roles/dataform.codeEditor
gcloud projects add-iam-policy-binding $PROJECT_ID --member serviceAccount:$CLOUD_BUILD_SERVICE_ACCOUNT --role roles/compute.admin
### enable the required APIs
BigQuery
Done
ENABLE NOW
ENABLE
ENABLE ALL
NEXT
NEXT
ENABLE ALL
### create a bigquery dataset for your project
BigQuery
Explorer
three dots
Create dataset

Dataset ID	        ecommerce
Location type	    select Multi-Region

Create Dataset
### Create a new python notebook and connect to the runtime
BigQuery
Create Python Notebook
REGION
Select

Connect to a runtime
Create a new runtime
Create default runtime
OPEN
Qwiklabs student ID
Allow
### Build the Python Notebook
- import Python libraries
from google.cloud import bigquery
from google.cloud import aiplatform
import bigframes.pandas as bpd
import pandas as pd
from vertexai.language_models._language_models import TextGenerationModel
from bigframes.ml.cluster import KMeans
from bigframes.ml.model_selection import train_test_split
- define variables and initiate the bigquery and vertex AI connection
project_id = 'qwiklabs-gcp-02-bb5d1a51dd96'
dataset_name = "ecommerce"
model_name = "customer_segmentation_model"
table_name = "customer_stats"
location = "us-central1"
client = bigquery.Client(project=project_id)
aiplatform.init(project=project_id, location=location)
- create and import the ecommerce.customer_stats table
%%bigquery
CREATE OR REPLACE TABLE ecommerce.customer_stats AS
SELECT
  user_id,
  DATE_DIFF(CURRENT_DATE(), CAST(MAX(order_created_date) AS DATE), day) AS days_since_last_order, ---RECENCY
  COUNT(order_id) AS count_orders, --FREQUENCY
  AVG(sale_price) AS average_spend --MONETARY
  FROM (
      SELECT
        user_id,
        order_id,
        sale_price,
        created_at AS order_created_date
        FROM `bigquery-public-data.thelook_ecommerce.order_items`
        WHERE
        created_at
            BETWEEN '2022-01-01' AND '2023-01-01'
  )
GROUP BY user_id;
- create a bigquery dataframe and load the data using a gimini prompt
import pandas as pd
df = bpd.read_gbq(f"{project_id}.{dataset_name}.{table_name}")
df.head(10)
- Generate the K-means clustering model
df_train, df_test = train_test_split(df, test_size=0.2,  random_state=42)
model = KMeans(n_clusters=5)
model.fit(df_train)
model.to_gbq(f"{project_id}.{dataset_name}.{model_name}")
- Create a visualization of the K-means clustering model results
import matplotlib.pyplot as plt
plt.scatter(predictions_df['days_since_last_order'], 
predictions_df['average_spend'], 
c=predictions_df['CENTROID_ID'])
plt.xlabel("days_since_last_order")
plt.ylabel("average_spend")
plt.title("Attribute grouped by K-means Cluster")
plt.show()
- Summarize each cluster generated from the K-means model
query = """
SELECT
 CONCAT('cluster ', CAST(centroid_id as STRING)) as centroid,
 average_spend,
 count_orders,
 days_since_last_order
FROM (
 SELECT centroid_id, feature, ROUND(numerical_value, 2) as value
 FROM ML.CENTROIDS(MODEL `{0}.{1}`)
)
PIVOT (
 SUM(value)
 FOR feature IN ('average_spend',  'count_orders', 'days_since_last_order')
)
ORDER BY centroid_id
""".format(dataset_name, model_name)
df_centroid = client.query(query).to_dataframe()
df_centroid.head()

df_query = client.query(query).to_dataframe()
df_query.to_string(header=False, index=False)
cluster_info = []
for i, row in df_query.iterrows():
 cluster_info.append("{0}, average spend ${2}, count of orders per person {1}, days since last order {3}"
  .format(row["centroid"], row["count_orders"], row["average_spend"], row["days_since_last_order"]) )
cluster_info = (str.join("\n", cluster_info))
print(cluster_info)
- Define a prompt for the marketing campaign
prompt = f"""
You're a creative brand strategist, given the following clusters, come up with \
creative brand persona, a catchy title, and next marketing action, \
explained step by step.
Clusters:
{cluster_info}
For each Cluster:
* Title:
* Persona:
* Next marketing step:
"""
- Generate the marketing campaign using the text-bison model
#prompt:  Use the Vertex AI language_models API to call the PaLM2 text-bison model and generate a marketing campaign using the variable prompt. Use the following model settings: max_output_tokens=1024, temperature=0.4

model = TextGenerationModel.from_pretrained("text-bison")
response = model.predict(prompt, max_output_tokens=1024, temperature=0.4)
print(response.text)
### clean up project resources
Manage resources
Delete
Shut Down Anyway
- clean up resources by deleting individual resources
  - Delete customer_stats table
  client.delete_table(f"{project_id}.{dataset_name}.{table_name}", not_found_ok=True)
  print(f"Deleted table: {project_id}.{dataset_name}.{table_name}")

  - Delete K-means model
  client.delete_model(f"{project_id}.{dataset_name}.{model_name}", not_found_ok=True)
  print(f"Deleted model: {project_id}.{dataset_name}.{model_name}")

## Designing an LLM connected model with Gemini
## Quiz
### how can you use gemini in bigquery to analyze data?
- complete a sql query
- explain how a sql query works
- generate sql queries to analyze your data from a description you write in english
### which vertex ai model can be used to identify customer clusters or segments?
- K-means clustering model
