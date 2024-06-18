# Detect Manufacturing Defects using Visual Inspection AI
## Create a Component Anomaly Detection Model using Visual Inspection AI [GSP895]
### Identify and detect components
- Create a dataset
Visual Inspection AI
Enable Visual Inspection AI API
Create a Dataset
Create Dataset
pcb_component
Assembly Inspection
Create

- Import training images into the dataset

export PROJECT_ID=$(gcloud config get-value core/project)
gsutil mb gs://${PROJECT_ID}
gsutil -m cp gs://cloud-training/gsp895/pcb_images/*.png \
gs://${PROJECT_ID}/demo_pcb_images/

gsutil ls gs://${PROJECT_ID}/demo_pcb_images/*.png > /tmp/demo_pcb_images.csv
gsutil cp /tmp/demo_pcb_images.csv gs://${PROJECT_ID}/demo_pcb_images.csv

Import
Select an import file from Cloud Storage
Browse
demo_pcb_image.csv
Select
Continue
Import in progress

- Configure a template
Components
select one image
Use as template
click back arrow 
click the template image
Add bounding box
Inspection Area: Included
Save

- Detect and crop components via alignment
@ Component tab
Add New Component
Resistance
Done

Template image
Add bounding box
draw a region for one Resistance component

repeat for another component

Save

@ Component tab
Detect Components

### Apply alignment and detection of components
Next
Apply to all
Apply
Apply to all task

### Training and refining an anomaly detection model


### Evaluating a trained Assembly Inspection model


### Creating a trained Component Assembly Detection solution artifact
Test & Use
Create Solution Artifact
CREATE

### Performing batch predictions using an Assembly Inspection solution artifact
Test & Use
Create Batch Prediction
CREATE

## Deploy and Test Visual Inspection AI Component Anomaly Detection Solution [GSP896]
### Deploy the exported Assembly Inspection solution artifact
- Run and test a CPU based solution artifact locally

gcloud compute ssh vm_name --zone vm_zone

export DOCKER_TAG=gcr.io/ql-shared-resources-test/resistance_solution@sha256:d9095cbd6f7ca69b1a30c58c4272b68062d2004ed259ff0dcb9af0ceb92b393b

export VISERVING_CPU_DOCKER_WITH_MODEL=${DOCKER_TAG}
export HTTP_PORT=8602
export LOCAL_METRIC_PORT=8603

docker pull ${VISERVING_CPU_DOCKER_WITH_MODEL}

docker run -v /secrets:/secrets --rm -d --name "test_cpu" \
--network="host" \
-p ${HTTP_PORT}:8602 \
-p ${LOCAL_METRIC_PORT}:8603 \
-t ${VISERVING_CPU_DOCKER_WITH_MODEL} \
--metric_project_id="${PROJECT_ID}" \
--use_default_credentials=false \
--service_account_credentials_json=/secrets/assembly-usage-reporter.json

docker container ls 

gsutil cp gs://cloud-training/gsp895/prediction_script.py .

### Serve the exported Assembly Inspection solution artifact
- Identifying a defective component

export PROJECT_ID=$(gcloud config get-value core/project)
gsutil mb gs://${PROJECT_ID}
gsutil -m cp gs://cloud-training/gsp895/pcb_images/*.png \
gs://${PROJECT_ID}/demo_pcb_images/
gsutil cp gs://${PROJECT_ID}/demo_pcb_images/image_275_cx98_cy16_r-5.png .

python3 ./prediction_script.py --input_image_file=./image_275_cx98_cy16_r-5.png  --port=8602 --output_result_file=def_prediction_result.json

python3 ./prediction_script.py --input_image_file=./image_275_cx98_cy16_r-5.png  --port=8602 --num_of_requests=10 --output_result_file=def_latency_result.json

- Identifying a non-defective component

export PROJECT_ID=$(gcloud config get-value core/project)
gsutil cp gs://${PROJECT_ID}/demo_pcb_images/image_439_cx31_cy-35_r-4.png .

python3 ./prediction_script.py --input_image_file=./image_439_cx31_cy-35_r-4.png  --port=8602 --output_result_file=non_def_prediction_result.json

python3 ./prediction_script.py --input_image_file=./image_275_cx98_cy16_r-5.png  --port=8602 --num_of_requests=10 --output_result_file=non_def_latency_result.json

## Create a Cosmetic Anomaly Detection Model using Visual Inspection AI [GSP897]
### Create a dataset
Visual Inspection AI
Enable Visual Inspection AI API

Create a Dataset
Create Dataset
cosmetic
Cosmetic Inspection
Polygon
US Central1
Create

### Import training images into the dataset

export PROJECT_ID=$(gcloud config get-value core/project)
gsutil mb gs://${PROJECT_ID}
gsutil -m cp gs://cloud-training/gsp897/cosmetic-test-data/*.png \
gs://${PROJECT_ID}/cosmetic-test-data/

gsutil ls gs://${PROJECT_ID}/cosmetic-test-data/*.png > /tmp/demo_cosmetic_images.csv
gsutil cp /tmp/demo_cosmetic_images.csv gs://${PROJECT_ID}/demo_cosmetic_images.csv

Import
Select an import method
Select an import file from Cloud Storage
Import File Path
Browse
demo_cosmetic_images.csv
Select
Continue

### Provide annotation for defect instances in training images
Defects
Add New Defect Type
Defect type:    dent
Done
Create
Add New Defect Type
Defect type:    scratch
Done
Create

Add Simple Polygon
Save

No defect
Confirm
No defect


## Deploy and Test a Visual Inspection AI Cosmetic Anomaly Detection Solution [GSP898]
### Deploy the exported Cosmetic Inspection anomaly detection solution artifact
gcloud compute ssh vm_name --zone vm_zone

export DOCKER_TAG=gcr.io/ql-shared-resources-test/defect_solution@sha256:776fd8c65304ac017f5b9a986a1b8189695b7abbff6aa0e4ef693c46c7122f4c

export VISERVING_CPU_DOCKER_WITH_MODEL=${DOCKER_TAG}
export HTTP_PORT=8602
export LOCAL_METRIC_PORT=8603

docker pull ${VISERVING_CPU_DOCKER_WITH_MODEL}

docker run -v /secrets:/secrets --rm -d --name "test_cpu" \
--network="host" \
-p ${HTTP_PORT}:8602 \
-p ${LOCAL_METRIC_PORT}:8603 \
-t ${VISERVING_CPU_DOCKER_WITH_MODEL} \
--metric_project_id="${PROJECT_ID}" \
--use_default_credentials=false \
--service_account_credentials_json=/secrets/assembly-usage-reporter.json

docker container ls

gsutil cp gs://cloud-training/gsp895/prediction_script.py .

### Serve the exported assembly inspection solution artifact
- Identifying a defective product

export PROJECT_ID=$(gcloud config get-value core/project)
gsutil mb gs://${PROJECT_ID}
gsutil -m cp gs://cloud-training/gsp897/cosmetic-test-data/*.png \
gs://${PROJECT_ID}/cosmetic-test-data/
gsutil cp gs://${PROJECT_ID}/cosmetic-test-data/IMG_07703.png .

python3 ./prediction_script.py --input_image_file=./IMG_07703.png  --port=8602 --output_result_file=def_prediction_result.json

python3 ./prediction_script.py --input_image_file=./IMG_07703.png  --port=8602 --num_of_requests=10 --output_result_file=def_latency_result.json

- Identifying a non-defective product

export PROJECT_ID=$(gcloud config get-value core/project)
gsutil cp gs://${PROJECT_ID}/cosmetic-test-data/IMG_0769.png .

python3 ./prediction_script.py --input_image_file=./IMG_0769.png  --port=8602 --output_result_file=non_def_prediction_result.json

python3 ./prediction_script.py --input_image_file=./IMG_0769.png  --port=8602 --num_of_requests=10 --output_result_file=non_def_latency_result.json

## Detect Manufacturing Defects using Visual Inspection AI: Challenge Lab [GSP366]
Challenge scenario
You are part of an international mobile manufacturing organization and tasked with inspecting defects on the surface of the mobile phones that are out from the assembly line. Your manager has asked you to help develop a system that will be used to analyze images taken on the production like to identify defects such as scratches, dents, deformations, etc. using a pre-prepared Google Visual Inspection Cosmetic Inspection model. The Visual Inspection model and solution artifact have been prepared by another member of your team and reside in shared resource project that you have access to.

Your challenge is to deploy the solution artifact and test that it can successfully identify both mobile phones that have cosmetic defects (defective mobile phones) and mobile phones that do not have cosmetic defects (non-defective mobile phones).
### Deploy the exported Cosmetic Inspection anomaly detection solution artifact

gcloud compute ssh lab-vm --zone us-east1-d

export DOCKER_TAG=gcr.io/ql-shared-resources-test/defect_solution@sha256:776fd8c65304ac017f5b9a986a1b8189695b7abbff6aa0e4ef693c46c7122f4c

export VISERVING_CPU_DOCKER_WITH_MODEL=${DOCKER_TAG}
export HTTP_PORT=8602
export LOCAL_METRIC_PORT=8603

docker pull ${VISERVING_CPU_DOCKER_WITH_MODEL}

docker run -v /secrets:/secrets --rm -d --name "mobile_inspection" \
--network="host" \
-p ${HTTP_PORT}:8602 \
-p ${LOCAL_METRIC_PORT}:8603 \
-t ${VISERVING_CPU_DOCKER_WITH_MODEL} \
--metric_project_id="${PROJECT_ID}" \
--use_default_credentials=false \
--service_account_credentials_json=/secrets/assembly-usage-reporter.json

docker container ls

### Prepare resources to serve the exported assembly inspection solution artifact

gsutil cp gs://cloud-training/gsp895/prediction_script.py .

gsutil mb gs://qwiklabs-gcp-01-61d326c8074a
gsutil -m cp gs://cloud-training/gsp897/cosmetic-test-data/*.png \
gs://qwiklabs-gcp-01-61d326c8074a/cosmetic-test-data/

### Identify a defective product image

gsutil cp gs://qwiklabs-gcp-01-61d326c8074a/cosmetic-test-data/IMG_07703.png .

python3 ./prediction_script.py --input_image_file=./IMG_07703.png  --port=8602 --output_result_file=defective_product.json

python3 ./prediction_script.py --input_image_file=./IMG_07703.png  --port=8602 --num_of_requests=10 --output_result_file=defective_product.json

### Identify a non-defective product 

gsutil cp gs://qwiklabs-gcp-01-61d326c8074a/cosmetic-test-data/IMG_0769.png .

python3 ./prediction_script.py --input_image_file=./IMG_0769.png  --port=8602 --output_result_file=non_defective_product_result.json

