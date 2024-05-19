# Chaiyo GCP Season 4
# Baseline: Infrastructure

# CLI [command line interface]
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

# Cloud IAM [google cloud identity and access management]
- three basic roles [viewer, editor, owner]

# cloud monitoring
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
# cloud function [console]
# 

