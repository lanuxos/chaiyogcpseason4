# Google Cloud Computing Foundations: Infrastructure in Google Cloud - Locales

## where do i store the staff?
- connect to database
gcloud sql connect myinstance --user=root
- create database
CREATE DATABASE guestbook;
- insert table, records
```
USE guestbook;
CREATE TABLE entries (guestName VARCHAR(255), content VARCHAR(255),
    entryID INT NOT NULL AUTO_INCREMENT, PRIMARY KEY(entryID));
    INSERT INTO entries (guestName, content) values ("first guest", "I got here!");
INSERT INTO entries (guestName, content) values ("second guest", "Me too!");
```
- retrieve data
SELECT * FROM entries;

## there is api for that
- get sample code
gsutil cp gs://spls/gsp164/endpoints-quickstart.zip .
unzip endpoints-quickstart.zip
cd endppoints-quickstart
- deploy the endpoint config
cd scripts
./deploy_api.sh
- deploying the api backend
./deploy_app.sh ../app/app_template.yaml us-east4
- sending request to the api
./query_api.sh
./query_api.sh JFK
- tracking api activity
./generate_traffic.sh
- add q quota to the api
./deploy_api.sh ../openapi_with_ratelimit.yaml
./deploy_app.sh ../app/app_template.yaml us-east4
Navmenu > APIs & Services > Credentials > Create credentials > API key
export API_KEY=YOUR-API-KEY
./query_api_with_key.sh $API_KEY
./generate_traffic_with_key.sh $API_KEY
./query_api_with_key.sh $API_KEY
- create venv
sudo apt-get install -y virtualenv
python3 -m venv venv
source venv/bin/activate
- install pub/sub client library
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
- publish message
gcloud pubsub topics publish MyTopic --message "Hello"
- view messages
python subscriber.py $GOOGLE_CLOUD_PROJECT receive MySub

## you can't secure the cloud, right?
### user authentication: Identity Aware Proxy [IAM] [GSP499]
- deploy the application and protect it with IAM
gsutil cp gs://spls/gsp499/user-authentication-with-iap.zip .
unzip user-authentication-with-iap.zip
cd user-authentication-with-iap
cd 1-HelloWorld
sed -i 's/python37/python39/g' app.yaml
gcloud app deploy
gcloud app browse
Security > Identity-Aware Proxy
ENABLE API
GO TO IDENTITY-AWARE PROXY
CONFIGURE CONSENT SCREEN
Internal
Create
Save and Continue
Scopes
Save and Continue
Summary
Back to Dashboard
- disable Flex API
gcloud services disable appengineflex.googleapis.com
App Engine app
IAP
Turn On
- add account to IAP
App Engine app
Add Principal
email address
Cloud IAP > IAP-Secured Web App User
Save
- access user identity information
cd ~/user-authentication-with-iap/2-HelloUser
sed -i 's/python37/python39/g' app.yaml
gcloud app deploy
gcloud app browse
- use cryptographic verification
cd ~/user-authentication-with-iap/3-HelloVerifiedUser
sed -i 's/python37/python39/g' app.yaml
gcloud app deploy
