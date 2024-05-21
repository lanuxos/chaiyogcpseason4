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
