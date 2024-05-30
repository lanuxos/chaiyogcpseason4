# Monitor and Manage Google Cloud Resources
## Cloud IAM: Qwik Start
### explore the IAM console and project level roles
IAM & Admin > IAM
+GRANT ACCESS
Basic
CANCEL
### prepare a cloud storage bucket for access testing
- create a bucket
Cloud Storage > Buckets
+CREATE
globally unique name (create it yourself!) and click CONTINUE
CREATE
Confirm
- upload files
UPLOAD FILES
Rename
RENAME
### remove project access
IAM & Admin > IAM
Username 2 [click the pencil icon inline]
clicking the trashcan icon
SAVE
### add cloud storage permissions
IAM & Admin > IAM
+GRANT ACCESS 
New principals [Username]
Select a role
Cloud Storage > Storage Object Viewer
SAVE

## Cloud Monitoring: Qwik Start [GSP089]
### create a compute engine instance
Compute Engine > VM instances
Create instance
Name	lamp-1-vm
Region	REGION
Zone	ZONE
Series	E2
Machine type	e2-medium
Boot disk	Debian GNU/Linux 12 (bookworm)
Firewall	Check Allow HTTP traffic
Create
### add apache2 http server to your instance
- install apache2 server
sudo apt-get update
sudo apt-get install apache2 php7.0
Y
sudo service apache2 restart
- create a monitoring metrics scope
Navigation menu > View All Products > Monitoring
- install the monitoring and logging agents [Cloud Logging agent]
curl -sSO https://dl.google.com/cloudagents/add-google-cloud-ops-agent-repo.sh
sudo bash add-google-cloud-ops-agent-repo.sh --also-install
Y
sudo systemctl status google-cloud-ops-agent"*"
q
sudo apt-get update
### create an uptime check
In the Cloud Console
Uptime checks
Create Uptime Check
Protocol = HTTP
Resource Type = Instance
Instance = lamp-1-vm
Check Frequency = 1 minute
Continue
Continue
Continue
Title = Lamp Uptime Check
Test
Create
### create an alerting policy
Alerting
+Create Policy
Select a metric dropdown. Uncheck the Active
Type Network traffic in filter by resource and metric name
VM instance > Interface
Apply
Next
Threshold position
Threshold value = 500
Advanced Options > Retest window
Next
Notification Channels
Manage Notification Channels

Email = ADD NEW
Create Email Channel
Email Address field
Display name
Save

Go back to the previous Create alerting policy tab
Notification Channels 
Refresh icon 
Notification Channels 
Display name 
OK
Alert name = Inbound Traffic Alert
Next
Create Policy
### create a dashboard and chart
- create a dashboard
Dashboards
+Create Dashboard
Name = Cloud Monitoring LAMP Qwik Start Dashboard
- add chart
+ADD WIDGET
Visualization = Line
Name = CPU Load
Select a metric dropdown. Uncheck the Active
Type CPU load (1m) in filter by resource and metric name 
VM instance > Cpu
CPU load (1m) 
Apply

+ADD WIDGET
Visualization = Line
Received Packets
Select a metric dropdown. Uncheck the Active
Type Received packets in filter by resource and metric name 
VM instance > Instance
Received packets 
Apply
### view your logs
Logging > Logs Explorer
lamp-1-vm
Resource
VM Instance > lamp-1-vm
Apply
Stream logs
### check the uptime check results and triggered alerts
- Check the uptime
Monitoring > Uptime checks
Lamp Uptime Check
- Check if alerts have been triggered
Alerting

## Cloud Functions: Qwik Start - Console [GSP081]
### create a function
Cloud Functions
Create function

Environment     2nd Gen
Function name   GCFunction
Region          REGION
Trigger type    HTTPS
Authentication  Allow unauthenticated invocations
Autoscaling     Set the Maximum number of instance to 5 

Next

### deploy the function
Deploy
Deploy
### test the function
Cloud Functions Overview 
GCFunction
TESTING
Triggering event = {"message":"Hello World!"}
### view logs
Cloud Functions Overview
three dots
View logs

## Monitor and Manage Google Cloud Resources: Challenge Lab [ARC101]
Challenge scenario
You are just starting your junior cloud engineer role. So far you have been helping teams create and manage Google Cloud resources.
You are expected to have the skills and knowledge for these tasks.

f1-micro for small linux vms
e2-medium for Windows

### Create a bucket
- create a bucket
Cloud Storage > Buckets
+CREATE
globally unique name (create it yourself!) and click CONTINUE
CREATE
Confirm
- grant storage object viewer to user 2
IAM & Admin > IAM
+GRANT ACCESS 
New principals [Username 2]
Select a role
Cloud Storage > Storage Object Viewer
SAVE
### Create a Pub/Sub topic
- create a Pub/Sub topic called [console]
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
### Create the thumbnail Cloud Function
- create a cloud function called [travel-thumbnail-maker]
Cloud Functions
Create function

Environment     1nd Gen
Function name   travel-thumbnail-maker
Region          REGION
Trigger type    HTTPS
Authentication  Allow unauthenticated invocations
Autoscaling     Set the Maximum number of instance to 5 

Next

Deploy
Deploy
- set Entry point to [thumbnail] and Trigger to Cloud Storage
- replace topic name [travel-topic-264] in line 15 of index.js
- upload image to bucket

### Create an alerting policy
- create an alerting policy called Active Cloud Function Instances
Alerting
+Create Policy
Select a metric dropdown. Uncheck the Active
Cloud Function > Function > Active Instances
Apply
Next

Threshold position
Threshold value = 0
Advanced Options > Retest window = 1
Next
Notification Channels
Manage Notification Channels

Email = ADD NEW
Create Email Channel
Email Address field
Display name
Save

Go back to the previous Create alerting policy tab
Notification Channels 
Refresh icon 
Notification Channels 
Display name 
OK
Alert name = Active Cloud Function Instances
Next
Create Policy
