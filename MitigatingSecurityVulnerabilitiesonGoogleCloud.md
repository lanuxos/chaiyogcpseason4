# MitigatingSecurityVulnerabilitiesonGoogleCloud
## Protecting against Distributed Denial of Service Attacks (DDoS)
### Module Overview
- 
### How DDos attacks work
- 
### DDoS mitigation and prevention on Google Cloud
- load balancer
- Cloud CDN
- Google Cloud Armor
### Using Google Cloud Armor
- 
### Types of complementary partner products
- 
### Lab Intro: Configureing Traffic Blocklisting with Google Cloud Armor
- 
### Configuring Traffic Blocklisting with Google Cloud Armor
- Verify the HTTP load balancer is deployed
gcloud compute backend-services get-health web-backend --global
- Retrieve the load balancer IP address
gcloud compute forwarding-rules describe web-rule --global
- test using curl
while true; do curl -m1 [IP_ADDRESS_OF_LOAD_BAL]; done
- Create a VM to test access to the load balancer
Compute Engine
Create Instance
Name:       access-test
Region:     australia-southeast1
Create
SSH

curl -m1 [IP_ADDRESS_OF_LOAD_BAL]

- Create a security policy with Google Cloud Armor
    - Blocklist the access-test VM
Compute engine
access-test
Network interface
External IP address

Network Security
Cloud Armor policies
Create policy
Name:       blocklist-access-test
Default rule action:    Allow
Next step
Add rule

Mode            Basic mode (IP addresses/ranges only)
Match           Enter the External IP of the access-test VM
Action          Deny
Response code   404 (Not Found)
Priority        1000

Done
Next step
+ Add Target
Type 1:                         Load balancer backend service
Backend Service target 1:       web-backend
Next step
Done
Create policy

    - Verify the security policy
SSH into access-test VM
curl -m1 [IP_ADDRESS_OF_LOAD_BAL]

- View Google CloudArmor logs
Network Secuiry
Cloud Armor policies
blocklist-access-test
Logs
View policy logs
404
httpRequest
access-test

### Module review
- 
### Quiz: Protecting against DDoS Attacks
- 

## Content-Related Vulnerabilities: Techniques and Best Practices
### Module overview
- 
### Threat: Ramsomware
- 
### Ransomware mitigations
- 
### Threat: Data misuse, privacy violations, sensitive content
- 
### Content-related mitigation
- 
### Lab Intro: Redacting Sensitive Data with the DLP API
- enable DLP API
- Install the DLP API and Node JS samples 
export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID

git clone https://github.com/GoogleCloudPlatform/nodejs-docs-samples

cd nodejs-docs-samples/dlp

npm install @google-cloud/dlp
npm install yargs
npm install mime

- Inspect and redact sensitive data
    - inspect a string for sensitive information
node inspectString.js $GCLOUD_PROJECT "My email address is joe@example.com."
node inspectString.js $GCLOUD_PROJECT "My phone number is 555-555-5555."
node deidentifyWithMask.js $GCLOUD_PROJECT "My phone number is 555-555-5555."
    - upload image
node redactImage.js $GCLOUD_PROJECT ~/dlp-input.png "" EMAIL_ADDRESS ~/dlp-redacted.png
- 
### Redacting Sensitive Data with the DLP API
- 

### Module review

### Quiz: Content Related Vulnarabilities
- 


## Monitoring Logging, Auditing and Scanning
### Module overview
- 
### Security Command Center
- 
### Tiers and pricing
- 
### Demo: Using Security Command Center
- 
### Cloud Monitoring and CLoud Logging
- 
### Lab Into: Configuring and Using Cloud Logging and Cloud Monitoring
- 
### Configuring and Using Cloud Logging and Cloud Monitoring
- Setup resources in your first project

curl https://storage.googleapis.com/cloud-training/gcpsec/labs/stackdriver-lab.tgz | tar -zxf -
cd stackdriver-lab

Open Editor 
Open in a new window
stackdriver-lab
linux_startup.sh
Replace the # install Ops Agent section with the following:

    # install Ops Agent
    curl -sSO https://dl.google.com/cloudagents/add-google-cloud-ops-agent-repo.sh
    sudo bash add-google-cloud-ops-agent-repo.sh --also-install
    Copied!
    After pasting, make sure that your lines of code are properly indented.

Save your file.

setup.sh
Update the image version in # create vms section for windows-server (row 17) after --image with the following:

    windows-server-2016-dc-core-v20240214
    
Add the following flag at the end of line 16 to set the machine type for the Linux VM:
--machine-type=e2-micro

Add the following flag at the end of line 17 to set the machine type for the Windows VM:
--machine-type=e2-standard-2

Save your file.
Open Terminal
replace zone with following command

sed -i 's/us-west1-b/lab zone/g' setup.sh
./setup.sh

- view and filter logs in first project
Logging
Logs Explorer

Log fields
Resource Type
VM Instance

- use log exports
    - Configure the export to BigQuery
Logging
Log Router
Create Sink
vm_logs
Next
BigQuery dataset
Create new BigQuery dataset
project_logs
Create Dataset
Next
resource.type="gce_instance"
Create Sink
    - Configure HTTP load balancing exports to BigQuery
Log Router
Create Sink
load_bal_logs
Next
BigQuery dataset
project_logs
Next
resource.type="http_load_balancer"
Create Sink
    - Investigate the exported log entries
BigQuery
Done
project_logs
Details

SELECT
  logName, resource.type, resource.labels.zone, resource.labels.project_id,
FROM
  `qwiklabs-gcp-xx.project_logs.syslog_xxxxx`

- create a logging metric
Logging
Logs Explorer
Create Metric
Counter
403s

resource.type="gce_instance"
log_name="projects/PROJECT_ID/logs/syslog"

Create Metric

- create a monitoring daskboard
Switch to project 2

Monitoring
Monitoring Settings
Add GCP Projects
Select Projects
Select
Add Projects
Dashboards
Create Dashboard
Add Widget 
Line
CPU Usage
Metric
Active
VM Instance
Instance
CPU Usage
Apply
Apply
Add Widget 
Line
Memory Utilization
Metric
Active
VM Instance
Instance
Memory
Memory Utilization
Apply
Apply

### Cloud Audit Logs
- 
### Lab Intro: Configuring and Viewing Cloud Audit Logs
- 
### Configuring and Viewing Cloud Audit Logs
- enable data access audit logs

gcloud projects set-iam-policy $DEVSHELL_PROJECT_ID ./policy.json

add following text to the policy.json

   "auditConfigs": [
      {
         "service": "allServices",
         "auditLogConfigs": [
            { "logType": "ADMIN_READ" },
            { "logType": "DATA_READ"  },
            { "logType": "DATA_WRITE" }
         ]
      }
   ],

gcloud projects set-iam-policy $DEVSHELL_PROJECT_ID ./policy.json

- generate some account activity
    - create a few source to generate activity
gsutil mb gs://$DEVSHELL_PROJECT_ID
echo "this is a sample file" > sample.txt
gsutil cp sample.txt gs://$DEVSHELL_PROJECT_ID
gcloud compute networks create mynetwork --subnet-mode=auto
gcloud compute instances create default-us-vm --machine-type=e2-micro --zone=us-central1-a --network=mynetwork
gsutil rm -r gs://$DEVSHELL_PROJECT_ID

- view the admin activity logs
Logging
Logs Explorer

logName = ("projects/[PROJECT_ID]/logs/cloudaudit.googleapis.com%2Factivity")

Run Query

or view via cloud shell with below command

gcloud logging read "logName=projects/$DEVSHELL_PROJECT_ID/logs/cloudaudit.googleapis.com%2Factivity AND protoPayload.serviceName=storage.googleapis.com AND protoPayload.methodName=storage.buckets.delete"

- export the audit logs
logName = ("projects/[PROJECT_ID]/logs/cloudaudit.googleapis.com%2Factivity")

Run Query
More actions
Create Sink
Sink Name:      AuditLogsExport
Next
Sink Service:   BigQuery dataset
Select Bigquery dataset:    Create new BigQuery dataset
Dataset ID:     auditLogs_dataset
Create Dataset
Uncheck Use Partitioned Tables
Next
logName = ("projects/[PROJECT_ID]/logs/cloudaudit.googleapis.com%2Factivity")
Create Sink
Logs Router
three dots
View sink details
Cancel

    - generate some more acivity
gsutil mb gs://$DEVSHELL_PROJECT_ID
gsutil mb gs://$DEVSHELL_PROJECT_ID-test
echo "this is another sample file" > sample2.txt
gsutil cp sample.txt gs://$DEVSHELL_PROJECT_ID-test
gcloud compute instances delete --zone=us-central1-a --delete-disks=all default-us-vm

gsutil rm -r gs://$DEVSHELL_PROJECT_ID
gsutil rm -r gs://$DEVSHELL_PROJECT_ID-test

- use bigquery to analyze logs
BigQuery
Done

    - generate some activity
gcloud compute instances create default-us-vm --zone=us-central1-a --network=mynetwork

gcloud compute instances delete --zone=us-central1-a --delete-disks=all default-us-vm

gsutil mb gs://$DEVSHELL_PROJECT_ID
gsutil mb gs://$DEVSHELL_PROJECT_ID-test
gsutil rm -r gs://$DEVSHELL_PROJECT_ID
gsutil rm -r gs://$DEVSHELL_PROJECT_ID-test

Query editor

#standardSQL
SELECT
  timestamp,
  resource.labels.instance_id,
  protopayload_auditlog.authenticationInfo.principalEmail,
  protopayload_auditlog.resourceName,
  protopayload_auditlog.methodName
FROM
`auditlogs_dataset.cloudaudit_googleapis_com_activity_*`
WHERE
  PARSE_DATE('%Y%m%d', _TABLE_SUFFIX) BETWEEN
  DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY) AND
  CURRENT_DATE()
  AND resource.type = "gce_instance"
  AND operation.first IS TRUE
  AND protopayload_auditlog.methodName = "v1.compute.instances.delete"
ORDER BY
  timestamp,
  resource.labels.instance_id
LIMIT
  1000

Run
replace with below query command

#standardSQL
SELECT
  timestamp,
  resource.labels.bucket_name,
  protopayload_auditlog.authenticationInfo.principalEmail,
  protopayload_auditlog.resourceName,
  protopayload_auditlog.methodName
FROM
`auditlogs_dataset.cloudaudit_googleapis_com_activity_*`
WHERE
  PARSE_DATE('%Y%m%d', _TABLE_SUFFIX) BETWEEN
  DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY) AND
  CURRENT_DATE()
  AND resource.type = "gcs_bucket"
  AND protopayload_auditlog.methodName = "storage.buckets.delete"
ORDER BY
  timestamp,
  resource.labels.instance_id
LIMIT
  1000

Run

### Cloud security automation
- 
### Module review
- 
### Module Quiz
- 



