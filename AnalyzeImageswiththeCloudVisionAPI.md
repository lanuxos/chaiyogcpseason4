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
### Create an API key
APIs & Services
Credentials
Create Credentials
API Key
Close

### Upload an image to a Cloud Vision API request
Cloud Storage
Create bucket
BUCKET_NAME
Choose how to control access to objects
Uncheck Enforce ;ublic access prevention on this bucket
Fine-grained
Create

upload sign.jpg
edit access [three dots]
add entry
Public
allUsers
Reader

### Create your Cloud Vision API request
ocr-request.json
```
{
  "requests": [
      {
        "image": {
          "source": {
              "gcsImageUri": "gs://BUCKET_NAME/sign.jpg"
          }
        },
        "features": [
          {
            "type": "TEXT_DETECTION",
            "maxResults": 10
          }
        ]
      }
  ]
}
```

### Call the text detection method
- call the text detection method
curl -s -X POST -H "Content-Type: application/json" --data-binary @ocr-request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY}

- save output of text detection method
curl -s -X POST -H "Content-Type: application/json" --data-binary @ocr-request.json  https://vision.googleapis.com/v1/images:annotate?key=${API_KEY} -o ocr-response.json

### Send text from the image to the Translation API
translation-request.json
```
{
  "q": "your_text_here",
  "target": "en"
}
```
- extract image text from previos and insert into translation-request.json
STR=$(jq .responses[0].textAnnotations[0].description ocr-response.json) && STR="${STR//\"}" && sed -i "s|your_text_here|$STR|g" translation-request.json

- call translation api
curl -s -X POST -H "Content-Type: application/json" --data-binary @translation-request.json https://translation.googleapis.com/language/translate/v2?key=${API_KEY} -o translation-response.json

### Analyzing the imange's text with the Natural Language API
nl-request.json
```
{
  "document":{
    "type":"PLAIN_TEXT",
    "content":"your_text_here"
  },
  "encodingType":"UTF8"
}
```
- copy the translaed text into nl-request.json
STR=$(jq .data.translations[0].translatedText  translation-response.json) && STR="${STR//\"}" && sed -i "s|your_text_here|$STR|g" nl-request.json

- call analyzeEntities endpoint of Natural Languages API
curl "https://language.googleapis.com/v1/documents:analyzeEntities?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @nl-request.json

## Detect Labels, Faces, and Landmarks in Images with the Cloud Vision API
### 

## Analyze Images with the Cloud Vision API: Challenge Lab
### 
