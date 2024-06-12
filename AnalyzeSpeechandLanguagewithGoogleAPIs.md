# Analyze Speech and Language with Google APIs
## Cloud Natural Language API: Qwik Start [GSP097]
### Create an API key
- export project id
export GOOGLE_CLOUD_PROJECT=$(gcloud config get-value core/project)

- create a new service account to access the Natural Language API
gcloud iam service-accounts create my-natlang-sa \
  --display-name "my natural language service account"

- Create credential key
gcloud iam service-accounts keys create ~/key.json \
  --iam-account my-natlang-sa@${GOOGLE_CLOUD_PROJECT}.iam.gserviceaccount.com

- export api key
export GOOGLE_APPLICATION_CREDENTIALS="/home/USER/key.json"

### Make an entity analysis request
Computer Engine
SSH

gcloud ml language analyze-entities --content="Michelangelo Caravaggio, Italian painter, is known for 'The Calling of Saint Matthew'." > result.json

## Speech-to-Text API: Qwik Start [GSP119]
### Create an api key
APIs & services
Credentials
Create credentials
API Key
Close

Compute Engine
VM instances
SSH

export API_KEY=

### create your speech-to-text api
request.json
```
{
  "config": {
      "encoding":"FLAC",
      "languageCode": "en-US"
  },
  "audio": {
      "uri":"gs://cloud-samples-tests/speech/brooklyn.flac"
  }
}
```
### call the speech-to-text-api
curl -s -X POST -H "Content-Type: application/json" --data-binary @request.json \
"https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}" > result.json

## Entity and Sentiment Analysis with the Natural Language API [GSP038]
### Create an API key
APIs & Services
Credentials
Create credentials
API key
Close

Compute Engine
VM instances
SSH

export API_KEY=

### Make an entity analysis request
nano request.json
```
{
  "document":{
    "type":"PLAIN_TEXT",
    "content":"Joanne Rowling, who writes under the pen names J. K. Rowling and Robert Galbraith, is a British novelist and screenwriter who wrote the Harry Potter fantasy series."
  },
  "encodingType":"UTF8"
}
```
### Call the Natural Language API
curl "https://language.googleapis.com/v1/documents:analyzeEntities?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @request.json > result.json

### Sentiment analysis with the Natural Language API
request.json
```
 {
  "document":{
    "type":"PLAIN_TEXT",
    "content":"Harry Potter is the best book. I think everyone should read it."
  },
  "encodingType": "UTF8"
}
```

curl "https://language.googleapis.com/v1/documents:analyzeSentiment?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @request.json

### Analyzing entity sentiment
request.json
```
 {
  "document":{
    "type":"PLAIN_TEXT",
    "content":"I liked the sushi but the service was terrible."
  },
  "encodingType": "UTF8"
}
```

curl "https://language.googleapis.com/v1/documents:analyzeEntitySentiment?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @request.json

### Analyzing syntax and parts of speech
request.json
```
{
  "document":{
    "type":"PLAIN_TEXT",
    "content": "Joanne Rowling is a British novelist, screenwriter and film producer."
  },
  "encodingType": "UTF8"
}
```
curl "https://language.googleapis.com/v1/documents:analyzeSyntax?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @request.json

### Multilingual natural language processing
request.json
```
{
  "document":{
    "type":"PLAIN_TEXT",
    "content":"日本のグーグルのオフィスは、東京の六本木ヒルズにあります"
  }
}
```

curl "https://language.googleapis.com/v1/documents:analyzeEntities?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @request.json

## Analyze Speech & Language with Google APIs: Challenge Lab [ARC114]

### Create an API key
### Make an entity analysis request and call the Natural Language API
### Create a speech analysis request and call the Speech API
### Analyze sentiment with the Natural Language API