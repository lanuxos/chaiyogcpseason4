# Analyze Images with the Cloud Vision API
## APIs Explore: Qwik Start [GSP277]
### Create a Cloud Storage bucket
Cloud Storage
Buckets
Create bucker
UNIQUE_BUCKET_NAME
Chose how to control access to objects
Uncheck Enforce publice acccess prevention on this bucket
Select Fine-grainted Access control
Create

### Upload an image
upload an image
Edit access [three dots]
+ Add entry
Public
allUsers
Reader

### Make a request to the Cloud Vision API service
APIs & Services
+ ENABLE APIS AND SERVICES
Cloud Vision
Enable
[Google Cloud Vision APIs](https://cloud.google.com/vision/docs/reference/rest/v1/images/annotate)

requests
features
type
LABEL_DETECTION
image
source
imageUri

```
{
    "request": [
        {
            "type": "LABEL_DETECTION"
        }
    ],
    "image": {
        "source": {
            "imageUri: "gs://BUCKET_NAME/IMAGE.jpg"
        }
    }
}
```
Google OAuth 2.0
API key
Execute

## Extract, Analyze, and Translate Text from Images with the Cloud ML APIs
### 

## Detect Labels, Faces, and Landmarks in Images with the Cloud Vision API
### 

## Analyze Images with the Cloud Vision API: Challenge Lab
### 
