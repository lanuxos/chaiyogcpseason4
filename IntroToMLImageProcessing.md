# Intro to ML: Image Processing

## Vertex AI Workbench Notebook: Qwik Start [GSP076]
### Launch Vertex AI Workbench notebook
Vertex AI
Workbench
Enable Notebooks API
User-Managed Notebooks
Create New
REGION
ZONE
TensorFlow Enterprise 2.11
Advanced Options
Machine type
e2-standard-2
Create
Open JupyterLab

### Clone the example repo within your Workbench instance

git clone --depth=1 https://github.com/GoogleCloudPlatform/training-data-analyst

training-data-analyst/self-paced-labs/ai-platform-qwikstart and open ai_platform_qwik_start.ipynb

### Run your training job in the cloud

### Test your understanding


## APIs Explorer: Qwik Start []


## Classify Images of Clouds in the Cloud with AutoML Images [GSP223]
### Set up AutoML
APIs & Services
Library
Cloud AutML
Enable
[open](https://console.cloud.google.com/vertex-ai)

- create storage bucket

gsutil mb -p $GOOGLE_CLOUD_PROJECT \
    -c standard    \
    -l us \
    gs://$GOOGLE_CLOUD_PROJECT-vcm/

### Upload training images to Cloud Storage
export BUCKET=$GOOGLE_CLOUD_PROJECT-vcm
gsutil -m cp -r gs://spls/gsp223/images/* gs://${BUCKET}

### Create a dataset
gsutil cp gs://spls/gsp223/data.csv .
sed -i -e "s/placeholder/${BUCKET}/g" ./data.csv
gsutil cp ./data.csv gs://${BUCKET}

Create Dataset
clouds
Image classification (Single-label)
Create
Select import files from Cloud Storage
Continue

### Inspect images


### Generate predictions
gsutil cp gs://spls/gsp223/examples/* .
cat CLOUD1-JSON
ENDPOINT=$(gcloud run services describe automl-service --platform managed --region REGION --format 'value(status.url)')
curl -X POST -H "Content-Type: application/json" $ENDPOINT/v1 -d "@${INPUT_DATA_FILE}" | jq

### Pop quiz
INPUT_DATA_FILE=CLOUD1-JSON
curl -X POST -H "Content-Type: application/json" $ENDPOINT/v1 -d "@${INPUT_DATA_FILE}" | jq

INPUT_DATA_FILE=CLOUD2-JSON
curl -X POST -H "Content-Type: application/json" $ENDPOINT/v1 -d "@${INPUT_DATA_FILE}" | jq

## Detect Labels, Faces, and Landmarks in Images with the Cloud Vision API []


## Extract, Analyze, and Translate Text from Images with the Cloud ML APIs []

