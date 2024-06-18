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


### Serve the exported Assembly Inspection solution artifact



## Create a Cosmetic Anomaly Detection Model using Visual Inspection AI []
 


## Deploy and Test a Visual Inspection AI Cosmetic Anomaly Detection Solution []
 


## Detect Manufacturing Defects using Visual Inspection AI: Challenge Lab []
 

