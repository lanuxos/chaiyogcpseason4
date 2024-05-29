# Chaiyo GCP Season 4
# Baseline: Infrastructure

## CLI [command line interface]
[gcloud cli overview guide](https://cloud.google.com/sdk/gcloud)
- list all the active account name
gcloud auth list
- to set active account, run
gcloud config set account `ACCOUNT`
- to list the project ID
gcloud config list project
- set project region
gcloud config set compute/region "REGION"
- make bucket
gsutil mb gs://BUCKET_NAME
- copy file
gsutil cp FILE gs://BUCKET_NAME
- remove file
rm FILE
- download file from bucket to command line
gsutil cp -r gs://BUCKET_NAME/FILE .
- copy file to specific directory
gsutil cp gs://BUCKET_NAME/ORIGINAL_FILE gs://BUCKET_NAME/DESTINATION_FOLDER/
- list contents of bucket or folder
gsutil ls gs://BUCKET_NAME
- list contents of bucket or folder with detail
gsutil ls -l gs://BUCKET_NAME/FILE
- make object publicly accessible [ACL - access control list]
gsutil acl ch -u AllUsers:R gs://BUCKET_NAME/FILE
- remove public access
gsutil acl ch -d AllUsers gs://BUCKET_NAME/FILE
- delete object
gsutil rm gs://BUCKET_NAME/FILE

## Cloud IAM [google cloud identity and access management]
- three basic roles [viewer, editor, owner]

## cloud monitoring
- setting up region and zone
gcloud config set compute/zone "ZONE"
export ZONE=$(gcloud config get compute/zone)
gcloud config set compute/region "REGION"
export REGION=$(gcloud config get compute/region)
- add apache2 http server to debian instance
sudo apt-get update && sudo apt-get install apache2 php7.0 -y
sudo service apache2 restart
- install monitoring agent script
curl -sSO https://dl.google.com/cloudagents/add-google-cloud-ops-agent-repo.sh
sudo bash add-google-cloud-ops-agent-repo.sh --also-install
- run logging agent install script command on VM
sudo systemctl status google-cloud-ops-agent"*"
sudo apt-get update
## cloud function [console]
## cloud function [command line]
- create function
gcloud config set compute/region us-central1
mkdir gcf_hello_world
cd gcf_hello_world
nano index.js
```
/**
* Background Cloud Function to be triggered by Pub/Sub.
* This function is exported by index.js, and executed when
* the trigger topic receives a message.
*
* @param {object} data The event payload.
* @param {object} context The event metadata.
*/
exports.helloWorld = (data, context) => {
const pubSubMessage = data;
const name = pubSubMessage.data
    ? Buffer.from(pubSubMessage.data, 'base64').toString() : "Hello World";

console.log(`My Cloud Function: ${name}`);
};
```
- create cloud storage bucket
gsutil mb -p PROJECT_ID gs://BUCKET_NAME
- deploy function
gcloud services disable cloudfunctions.googleapis.com
gcloud services enable cloudfunction.googleapis.com
gcloud projects add-iam-policy-binding PROJECT_ID \
--member="serviceAccount: PROJECT_ID@appspot.gserviceaccount.com" \
--role="roles/artifactregistry.reader"
- deploy function
gcloud functions deploy helloWorld \
--stage-bucket BUCKET_NAME \
--trigger-topic  hello_world
--runtime nodejs20
- verify function status
gcloud functions describe helloWorld
- test function
DATA=$(printf 'Hello World!'|base64) && gcloud functions call helloWorld --data '{"data":"'$DATA'"}'
- view logs
gcloud functions logs read helloWorld
## pub/sub [console]
- create topic
Pub/Sub > Topics
Create a topic
Topic ID
CREATE
- add a subscription
Topics
three dots
Create subscription
Add subscription to topic
Subscription ID = MySub
Pull
Create
- publish a message to the topic
Pub/Sub
Topics
MyTopics
Messages
Publish Message
Message = Hello World
Publish
- view the message
gcloud pubsub subscriptions pull --auto-ack MySub
## pub/sub [command line]
- topic
gcloud pubsub topics create myTopic
gcloud pubsub topics create Test1
gcloud pubsub topics create Test2
gcloud pubsub topics list
gcloud pubsub topics delete Test1
gcloud pubsub topics delete Test2
- subscription
gcloud pubsub subscriptions create --topic myTopic mySubscription
gcloud pubsub subscriptions create --topic myTopic Test1
gcloud pubsub subscriptions create --topic myTopic Test2
gcloud pubsub topics list-subscriptions myTopic
gcloud pubsub subscriptions delete Test1
gcloud pubsub subscriptions delete Test2
- publishing and pulling a single message
gcloud pubsub topics publish myTopic --message "Hello"
gcloud pubsub topics publish myTopic --message "World"
gcloud pubsub topics publish myTopic --message "Google"
gcloud pubsub topics publish myTopic --message "Cloud"
gcloud pubsub subscriptions pull mySubscription --auto-ack
- publishing and pulling multiple message
gcloud pubsub topics publish myTopic --message "Hello"
gcloud pubsub topics publish myTopic --message "World"
gcloud pubsub topics publish myTopic --message "Google"
gcloud pubsub topics publish myTopic --message "Cloud"
gcloud pubsub subscriptions pull mySubscription --auto-ack --limit=3
## pub/sub [python]
- install virtualenv
sudo apt-get install -y virtualenv
python3 -m venv venv
source venv/bin/activate
- install the client library
pip install --upgrade google-cloud-pubsub
git clone https://github.com/googleapis/python-pubsub.git
cd python-pubsub/samples/snippets
- create topic
echo $GOOGLE_CLOUD_PROJECT
cat publisher.py
python publisher.py -h
python publisher.py $GOOGLE_CLOUD_PROJECT create MyTopic
- create subscription
python subscriber.py $GOOGLE_CLOUD_PROJECT create MyTopic MySub
python subscriber.py $GOOGLE_CLOUD_PROJECT list-in-project
python subscriber.py -h
- publish message
gcloud pubsub topics publish MyTopic --message "Hello"
- view message
python subscriber.py $GOOGLE_CLOUD_PROJECT receive MySub
## pub/sub lite
- send message
```
from google.cloud.pubsublite.cloudpubsub import PublisherClient
from google.cloud.pubsublite.types import (
    CloudRegion,
    CloudZone,
    MessageMetadata,
    TopicPath,
)

# TODO(developer):
project_number = 29899377729
cloud_region = "europe-west1"
zone_id = "b"
topic_id = "my-lite-topic"
num_messages = 100

location = CloudZone(CloudRegion(cloud_region), zone_id)
topic_path = TopicPath(project_number, location, topic_id)

# PublisherClient() must be used in a `with` block or have __enter__() called before use.
with PublisherClient() as publisher_client:
    data = "Hello world!"
    api_future = publisher_client.publish(topic_path, data.encode("utf-8"))
    # result() blocks. To resolve API futures asynchronously, use add_done_callback().
    message_id = api_future.result()
    publish_metadata = MessageMetadata.decode(message_id)
    print(
        f"Published a message to partition {publish_metadata.partition.value} and offset {publish_metadata.cursor.offset}."
    )
```
- receive message
```
from concurrent.futures._base import TimeoutError
from google.cloud.pubsublite.cloudpubsub import SubscriberClient
from google.cloud.pubsublite.types import (
    CloudRegion,
    CloudZone,
    FlowControlSettings,
    SubscriptionPath,
)

# TODO(developer):
project_number = 29899377729
cloud_region = "europe-west1"
zone_id = "b"
topic_id = "my-lite-topic"
num_messages = 100

location = CloudZone(CloudRegion(cloud_region), zone_id)
subscription_path = SubscriptionPath(project_number, location, subscription_id)
# Configure when to pause the message stream for more incoming messages based on the
# maximum size or number of messages that a single-partition subscriber has received,
# whichever condition is met first.
per_partition_flow_control_settings = FlowControlSettings(
    # 1,000 outstanding messages. Must be >0.
    messages_outstanding=1000,
    # 10 MiB. Must be greater than the allowed size of the largest message (1 MiB).
    bytes_outstanding=10 * 1024 * 1024,
)

def callback(message):
    message_data = message.data.decode("utf-8")
    print(f"Received {message_data} of ordering key {message.ordering_key}.")
    message.ack()

# SubscriberClient() must be used in a `with` block or have __enter__() called before use.
with SubscriberClient() as subscriber_client:

    streaming_pull_future = subscriber_client.subscribe(
        subscription_path,
        callback=callback,
        per_partition_flow_control_settings=per_partition_flow_control_settings,
    )

    print(f"Listening for messages on {str(subscription_path)}...")

    try:
        streaming_pull_future.result(timeout=timeout)
    except TimeoutError or KeyboardInterrupt:
        streaming_pull_future.cancel()
        assert streaming_pull_future.done()
```
