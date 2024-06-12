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
APIs & Services
Credentials
Create credentials
API key
Close

Compute Engine
VM instances
SSH

export API_KEY=

### Make an entity analysis request and call the Natural Language API
nl_request.json
```
{
  "document":{
    "type":"PLAIN_TEXT",
    "content":"With approximately 8.2 million people residing in Boston, the capital city of Massachusetts is one of the largest in the United States."
  },
  "encodingType":"UTF8"
}
```

curl "https://language.googleapis.com/v1/documents:analyzeEntitySentiment?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @nl_request.json > nl_response.json

or

curl "https://language.googleapis.com/v1/documents:analyzeEntities?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @nl_request.json > nl_response.json
### Create a speech analysis request and call the Speech API
speech_request.json
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

curl -s -X POST -H "Content-Type: application/json" --data-binary @speech_request.json \
"https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}" > speech_response.json

or

curl -s -X POST -H "Content-Type: application/json" --data-binary @speech_request.json \
"https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}" > speech_response.json
### Analyze sentiment with the Natural Language API
edit sentiment_analysis.py
from
```
import argparse

from google.cloud import language_v1

def print_result(annotations):
    score = annotations.document_sentiment.score
    magnitude = annotations.document_sentiment.magnitude

    for index, sentence in enumerate(annotations.sentences):
        sentence_sentiment = sentence.sentiment.score
        print(
            "Sentence {} has a sentiment score of {}".format(index, sentence_sentiment)
        )

    print(
        "Overall Sentiment: score of {} with magnitude of {}".format(score, magnitude)
    )
    return 0

def analyze(movie_review_filename):
    """Run a sentiment analysis request on text within a passed filename."""


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description=__doc__, formatter_class=argparse.RawDescriptionHelpFormatter
    )
    parser.add_argument(
        "movie_review_filename",
        help="The filename of the movie review you'd like to analyze.",
    )
    args = parser.parse_args()

    analyze(args.movie_review_filename)
```
to
```
import argparse

from google.cloud import language_v1

def print_result(annotations):
    score = annotations.document_sentiment.score
    magnitude = annotations.document_sentiment.magnitude

    for index, sentence in enumerate(annotations.sentences):
        sentence_sentiment = sentence.sentiment.score
        print(
            f"Sentence {index} has a sentiment score of {sentence_sentiment}"
        )

    print(
        f"Overall Sentiment: score of {score} with magnitude of {magnitude}"
    )
    return 0


def analyze(movie_review_filename):
    """Run a sentiment analysis request on text within a passed filename."""
    client = language_v1.LanguageServiceClient()

    with open(movie_review_filename) as review_file:
        # Instantiates a plain text document.
        content = review_file.read()

    document = language_v1.Document(
        content=content, type_=language_v1.Document.Type.PLAIN_TEXT
    )
    annotations = client.analyze_sentiment(request={"document": document})

    # Print the results
    print_result(annotations)


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description=__doc__, formatter_class=argparse.RawDescriptionHelpFormatter
    )
    parser.add_argument(
        "movie_review_filename",
        help="The filename of the movie review you'd like to analyze.",
    )
    args = parser.parse_args()

    analyze(args.movie_review_filename)
```

gsutil cp gs://cloud-samples-tests/natural-language/sentiment-samples.tgz .

gunzip sentiment-samples.tgz
tar -xvf sentiment-samples.tar

python3 sentiment_analysis.py reviews/bladerunner-pos.txt