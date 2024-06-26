# Securing your Network with Cloud Armor
## Bot Management with Google Cloud Armor and reCAPTCHA [GSP877]
- set up project ID
export PROJECT_ID=$(gcloud config get-value project)
echo $PROJECT_ID
gcloud config set project $PROJECT_ID
- enable APIs
gcloud services enable compute.googleapis.com
gcloud services enable logging.googleapis.com
gcloud services enable monitoring.googleapis.com
gcloud services enable recaptchaenterprise.googleapis.com
### configure firewall rules to allow HTTP and SSH traffic to backends
- Create a firewall rule to allow HTTP traffic to the backends [console]
VPC network > Firewall
Create Firewall Rule

Name	            default-allow-health-check
Network	            default
Targets	            Specified target tags
Target tags	        allow-health-check
Source filter	    IPv4 Ranges
Source IPv4 ranges	130.211.0.0/22, 35.191.0.0/16
Protocols and ports	Specified protocols and ports, and then check tcp. Type 80 for the port number

- Create a firewall rule to allow HTTP traffic to the backends [cli]
gcloud compute firewall-rules create default-allow-health-check --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=allow-health-check

gcloud compute firewall-rules create allow-ssh --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:22 --source-ranges=0.0.0.0/0 --target-tags=allow-health-check

### configure instance templates and create managed instance groups
- Configure the instance templates
Compute Engine
Instance templates
Create instance template

Name        lb-backend-template
Location    Global
Series      N1
Advanced options
Networking, Disks, Security, Management, Sole-Tenancy 
Management
Startup script

```
#! /bin/bash
sudo apt-get update
sudo apt-get install apache2 -y
sudo a2ensite default-ssl
sudo a2enmod ssl
sudo su
vm_hostname="$(curl -H "Metadata-Flavor:Google" \
http://metadata.google.internal/computeMetadata/v1/instance/name)"
echo "Page served from: $vm_hostname" | \
tee /var/www/html/index.html
```

Networking : allow-health-check
Network (Under Network Interfaces)	    default
Subnetwork (Under Network Interfaces)	default (region)
Network tags	                        allow-health-check

Create

- Create the managed instance group
Compute Engine 
Instance groups
Create instance group
New managed instance group (stateless)

Name	                    lb-backend-example
Location	                Single zone
Region	                    region
Zone	                    zone
Instance template	        lb-backend-template
Autoscaling	                Set Autoscaling mode to Off: do not autoscale
Minimum number of instance	1

Create

- Add a named port to the instance group
gcloud compute instance-groups set-named-ports lb-backend-example --named-ports http:80 --zone us-east4-b

### configure the HTTP load balancer
- Start the configuration
Network Services
Load balancing
Create load balancer
Application Load Balancer (HTTP/HTTPS)
Next
Public facing (external)
Next
Best for global workloads 
Next
Global external Application Load Balancer 
Next
Configure
Name : http-lb
- Configure the frontend
Frontend configuration

Protocol	    HTTP
IP version	    IPv4
IP address	    Ephemeral
Port	        80

Done
- Configure the backend
Backend configuration
Create a backend service

Name	        http-backend
Protocol	    HTTP
Named Port	    http
Instance group	lb-backend-example
Port numbers	80

Done
Cloud CDN
Cache mode : Use origin settings based on Cache-Control headers
Health Check : Create a health check

Name	    http-health-check
Protocol	TCP
Port	    80

Save
Enable Logging box
Sample rate : 1
Create 
ok

- Review and create the HTTP Load Balancer
Review and finalize
Backend services and Frontend
Create

http-lb
NOTE IP 
- Test the HTTP Load Balancer
http://IP

### create and deploy reCAPTCHA session token and challenge page site key
- Create reCAPTCHA session token and WAF challenge-page site key
  - Create the reCAPTCHA session token site key and enable the WAF feature for the key
gcloud recaptcha keys create --display-name=test-key-name --web --allow-all-domains --integration-type=score --testing-score=0.5 --waf-feature=session-token --waf-service=ca
  - Create the reCAPTCHA WAF challenge-page site key and enable the WAF feature for the key
gcloud recaptcha keys create --display-name=challenge-page-key --web --allow-all-domains --integration-type=INVISIBLE --waf-feature=challenge-page --waf-service=ca
  - check created keys
  Security 
  reCAPTCHA Enterprise
  Enterprise Keys
Compute Engine > VM Instances
- Implement reCAPTCHA session token site key
Compute Engine > VM Instances > SSH

cd /var/www/html/
sudo su

  - update index.html
src="https://www.google.com/recaptcha/enterprise.js?render=<SESSION_TOKEN_SITE_KEY>&waf=session" async defer>

echo '<!doctype html><html><head><title>ReCAPTCHA Session Token</title><script src="https://www.google.com/recaptcha/enterprise.js?render=6LeNnu8pAAAAAEsxD2WcREf6hqeBlfRhnXWYl9Gb&waf=session" async defer></script></head><body><h1>Main Page</h1><p><a href="/good-score.html">Visit allowed link</a></p><p><a href="/bad-score.html">Visit blocked link</a></p><p><a href="/median-score.html">Visit redirect link</a></p></body></html>' > index.html

echo '<!DOCTYPE html><html><head><meta http-equiv="Content-Type" content="text/html; charset=windows-1252"></head><body><h1>Congrats! You have a good score!!</h1></body></html>' > good-score.html

echo '<!DOCTYPE html><html><head><meta http-equiv="Content-Type" content="text/html; charset=windows-1252"></head><body><h1>Sorry, You have a bad score!</h1></body></html>' > bad-score.html

echo '<!DOCTYPE html><html><head><meta http-equiv="Content-Type" content="text/html; charset=windows-1252"></head><body><h1>You have a median score that we need a second verification.</h1></body></html>' > median-score.html

### create cloud armor security policy rules for bot management 
- create security policy 
gcloud compute security-policies create recaptcha-policy --description "policy for bot management"
- use reCAPTCHA Enterprise
gcloud compute security-policies update recaptcha-policy --recaptcha-redirect-site-key "6Ld8b-8pAAAAAJtwfj9aXsVTYFqba5jovy3VLwB9"
- add a bot management rule to allow traffic if the url path matches good-score.html and has a score greater than 0.4
gcloud compute security-policies rules create 2000 --security-policy recaptcha-policy --expression "request.path.matches('good-score.html') && token.recaptcha_session.score > 0.4" --action allow
- add a bot management rule to deny traffic if the url path matches bad-score.html and has a score less than 0.6
gcloud compute security-policies rules create 3000 --security-policy recaptcha-policy --expression "request.path.matches('bad-score.html') && token.recaptcha_session.score < 0.6" --action "deny-403"
- Add a bot management rule to redirect traffic to Google reCAPTCHA if the url path matches median-score.html and has a score equal to 0.5
gcloud compute security-policies rules create 1000 --security-policy recaptcha-policy --expression "request.path.matches('median-score.html') && token.recaptcha_session.score == 0.5" --action redirect --redirect-type google-recaptcha
- Attach the security policy to the backend service http-backend
gcloud compute backend-services update http-backend --security-policy recaptcha-policy --global

### validate bot management with cloud armor
- verify cloud armor logs
Network Security 
Cloud Armor
recaptcha-policy
Logs
View policy logs
- MQL (monitoring query language)
resource.type:(http_load_balancer) AND jsonPayload.enforcedSecurityPolicy.name:(recaptcha-policy)

## Rate Limiting with Cloud Armor [GSP975]
### configure http and health check firewall rules
- create the http firewall rule
VPC network
Firewall
ICMP, internal, RDP and SSH
Create Firewall Rule

Name	                default-allow-http
Network	              default
Targets	              Specified target tags
Target tags	          http-server
Source filter	        IPv4 Ranges
Source IP ranges	    0.0.0.0/0
Protocols and ports	  Specified protocols and ports, and then check tcp, type: 80

Create

- Create the health check firewall rules
Create Firewall Rule

Name	                default-allow-health-check
Network	              default
Targets	              Specified target tags
Target tags	          http-server
Source filter	        IPv4 Ranges
Source IP ranges	    130.211.0.0/22, 35.191.0.0/16
Protocols and ports	  Specified protocols and ports, and then check tcp

Create

### configure instance templates and create instance groups
- configure the instance templates
Compute Engine
Instance templates
Create instance template

Name        REGION-template
Location    Global
Series      E2
Click Networking, disks, security, management, sole-tenancy
Management
Metadata > +ADD ITEM

Key         startup-script-url
Value       gs://cloud-training/gcpnet/httplb/startup.sh

Networking
Network tags  http-server
Network interfaces

Network	default
Subnet	default(REGION)

Create

REGION-template 
CREATE SIMILAR 
Name      REGION 3-template
Location  Global
Networking, disks, security, management, sole-tenancy
Networking, expand default network
Subnet    default (REGION 3)
Create

- Create the managed instance groups
Compute Engine
Instance groups
Create instance group

Name	                        REGION-mig
Location	                    Multiple zones
Region	                      REGION
Instance template	            REGION-template
Autoscaling > Autoscaling signals (click the dropdown icon to edit) > Signal type	CPU utilization
Target CPU utilization	      80, click Done.
Initialization period	        45
Minimum number of instances	  1
Maximum number of instances	  5

Create

Compute Engine
Instance groups
Create instance group

Name	                        REGION 3-mig
Location	                    Multiple zones
Region	                      REGION 3
Instance template	            REGION 3-template
Autoscaling > Autoscaling signals (click the dropdown icon to edit) > Signal type	CPU utilization
Target CPU utilization	      80, click Done.
Initialization period	        45
Minimum number of instances	  1
Maximum number of instances	  5

Create

- Verify the backends
VM instances
REGION-mig and REGION 3-mig
External IP of an instance of REGION-mig
External IP of an instance of REGION 3-mig

### configure the http load balancer
- Start the configuration
Network Services
Load balancing
Create load balancer
Application Load Balancer (HTTP/S)
Next
Under Public facing or internal only
Public facing (external)
Next
Global or single region deployment
Best for global workloads
Next
Under Load balancer generation
Global external Application Load Balancer
Next
Configure.
Load Balancer Name : http-lb

- Configure the backend
Backend configuration
Backend services & backend buckets 
Create a backend service

Name	http-backend
Instance group	REGION-mig
Port numbers	80
Balancing mode	Rate
Maximum RPS	50
Capacity	100

Done
Add a backend

Instance group	REGION 3-mig
Port numbers	80
Balancing mode	Utilization
Maximum backend utilization	80
Capacity	100

Done
Health Check > Create a health check

Name	http-health-check
Protocol	TCP
Port	80

Save
Enable Logging box
Sample Rate to 1:
Create 
OK

- Configure the frontend
Frontend configuration

Protocol	HTTP
IP version	IPv4
IP address	Ephemeral
Port	80

Done
Add Frontend IP and port

Protocol	HTTP
IP version	IPv6
IP address	Auto-allocate
Port	80

Done

- Review and create the HTTP Load Balancer
name: http-lb

Create

### test the http load balancer
- Stress test the HTTP Load Balancer
Compute Engine
VM instances
Create instance

Name	siege-vm
Region	REGION 2
Zone	ZONE 2
Series	E2

Create

SSH 

sudo apt-get -y install siege

export LB_IP=35.186.239.46
[2600:1901:0:6366::]

siege -c 250 http://$LB_IP

Network Services
Load balancing
Backends
http-backend
http-lb
Monitoring

### create clound armor rate limiting policy
- create security policy via gcloud
gcloud compute security-policies create rate-limit-siege --description "policy for rate limiting"
- add rate limiting rule
gcloud beta compute security-policies rules create 100 --security-policy=rate-limit-siege     --expression="true" --action=rate-based-ban --rate-limit-threshold-count=50 --rate-limit-threshold-interval-sec=120 --ban-duration-sec=300 --conform-action=allow --exceed-action=deny-404 --enforce-on-key=IP
- attach the security policy to the backend service http-backend
gcloud compute backend-services update http-backend --security-policy rate-limit-siege --global

Network Security > Cloud Armor
rate-limit-siege

### verify the security policy
- run a curl against the LB IP to verify you can still connect
curl http://$LB_IP
- simulate a load
siege -c 250 http://$LB_IP
- explore the security policy logs to determine if this traffic is also blocked
Network Security
Cloud Armor policies.
rate-limit-siege
Logs
View policy logs
Application Load Balancer
http-lb-forwarding-rule
http-lb 
Apply
Run Query
httpRequest

## HTTP Load Balancer with Cloud Armor [GSP215]
### configure http and health check firewall rules
- create the http firewall rule
VPC network
Firewall
Create Firewall Rule

Name	default-allow-http
Network	default
Targets	Specified target tags
Target tags	http-server
Source filter	IPv4 Ranges
Source IPv4 ranges	0.0.0.0/0
Protocols and ports	Specified protocols and ports, and then check TCP, type: 80

Create

- Create the health check firewall rules
Create Firewall Rule

Name	default-allow-health-check
Network	default
Targets	Specified target tags
Target tags	http-server
Source filter	IPv4 Ranges
Source IPv4 ranges	130.211.0.0/22, 35.191.0.0/16
Protocols and ports	Specified protocols and ports, and then check TCP

Create

### configure instance templates and create instance groups
- Configure the instance templates
Compute Engine
Instance templates
Create instance template

Name      Region 1-template
Location      Global
Series      E2
Machine      select e2-micro
Advanced Options
Networking

Network tags	http-server

Network	default
Subnetwork	default Region 1

Done

Management
Metadata
ADD ITEM

startup-script-url	gs://cloud-training/gcpnet/httplb/startup.sh

Create

Region 1-template 
+CREATE SIMILAR option from the top.
For Name, type Region 2-template.
Ensure Location is selected Global.
Click Advanced Options.
Click Networking.
Ensure http-server is added as a network tag.
In Network interfaces, for Subnetwork, select default (Region 2).
Click Done.
Click Create

- Create the managed instance groups
Instance groups 
Create instance group

Name	Region 1-mig (if required, remove extra space from the name)
Instance template	Region 1-template
Location	Multiple zones
Region	Region 1
Minimum number of instances	1
Maximum number of instances	2
Autoscaling signals > Click dropdown > Signal type	CPU utilization
Target CPU utilization	80, click Done.
Initialization period	45

Create

Create Instance group

Name	Region 2-mig
Instance template	Region 2-template
Location	Multiple zones
Region	Region 2
Minimum number of instances	1
Maximum number of instances	2
Autoscaling signals > Click dropdown > Signal type	CPU utilization
Target CPU utilization	80, click Done.
Initialization period	45

Create

- Verify the backends
### configure the http load balancer
Network Services > Load balancing
click Create load balancer
Under Application Load Balancer HTTP(S), click Next
For Public facing or internal, select Public facing (external) and click Next
For Global or single region deployment, select Best for global workloads and click Next
For Create load balancer, click Configure
Set the New HTTP(S) Load Balancer Name to http-lb

- Configure the frontend
Frontend configuration

Protocol	HTTP
IP version	IPv4
IP address	Ephemeral
Port	80

Done

Add Frontend IP and port

Protocol	HTTP
IP version	IPv6
IP address	Auto-allocate
Port	80

Done

- Configure the backend
Backend configuration
Create a backend service

Name	http-backend
Instance group	Region 1-mig
Port numbers	80
Balancing mode	Rate
Maximum RPS	50
Capacity	100

Done

Add a backend

Instance group	Region 2-mig
Port numbers	80
Balancing mode	Utilization
Maximum backend utilization	80
Capacity	100

Done

Health Check
Create a health check

Name	http-health-check
Protocol	TCP
Port	80

Save.
Check the Enable Logging box.
Set the Sample Rate to 1.
Click Create to create the backend service.
Click Ok.

- Review and create the HTTP Load Balancer
name: http-lb
Create

### test the http load balancer
- Stress test the HTTP Load Balancer
Compute Engine
VM instances
Create instance

Name	siege-vm
Region	Region 3
Zone	Zone 3
Series	E2

Create

SSH

sudo apt-get -y install siege
export LB_IP=[LB_IP_v4]
siege -c 150 -t120s http://$LB_IP

### denylist the sighe-vm
- Create the security policy
Network Security
Cloud Armor Policies
Click Create policy

Name	denylist-siege
Default rule action	Allow

Next step
Click Add a rule

Condition > Match	Enter the SIEGE_IP
Action	Deny
Response code	403 (Forbidden)
Priority	1000

Done
Next step
Add Target
Type: Backend service (external application load balancer)
Target: http-backend
Create policy

## Defending Edge Cache with Cloud Armor [GSP878]
### create a cloud storage bucket and uplaod an object
Cloud Storage
Buckets
CREATE
BucketName
Continue
Location type:    REGION
Continue
The default storage class for your bucket:  standard
Continue
Uncheck Enforce public access prevention on this bucket
Check Prevent public access
Access Control:   Fine-grained
Continue
Create

- Upload an Object to the bucket
wget --output-document google.png https://www.google.com/images/branding/googlelogo/1x/googlelogo_color_272x92dp.png
gsutil cp google.png gs://Bucket Name
rm google.png

Cloud Storage
Buckets
BucketName
Exit access
Add Entry
Public
Save

Entity:   Public
Name:     allUsers
Access:   Reader

### create a load balancer
Network services
Load Balancing
CREATE LOAD BALANCER
load balancer:    Application Load Balancer (HTTP/HTTPS)
Next
Public facing or internal:    Public facing (external)
Next
Global or single region deployment:   Best for global workloads
Next
Load balancer generation:     Global external Application Load Balancer
Next
CONFIGURE
Name:   edge-cache-lb
- Frontend configuration
default
Done
- Create backend configuration
Backend configuration
Backend services & backend buckets:     Create a backend bucket
Backend bucket name:    lb-backend-bucket
Browse
Create
- Create host and path rules
Routing rules
simple host and path rule
- Review and create the HTTP Load Balancer
Review and finalize
Backend services and Frontend
Create
- Get Load Balancer IP
- Query the Load balancer
curl -svo /dev/null http://LOAD_BALANCER_IP/google.png
for i in `seq 1 50`; do curl http://LOAD_BALANCER_IP/google.png; done

### delete the object from cloud storage bucket
Cloud Storage
BucketName
Delete
Delete

### create an edge security policy
Cloud Armor policies
Create Policy

Name	edge-security-policy
Policy type	Edge security policy
Default rule action	Deny

Apply policy to targets section
Add Target

Type 1	Backend bucket (external application load balancer)
Backend Bucket target 1	lb-backend-bucket

Done
Create Policy

- Validate Edge Security Policy
curl -svo /dev/null http://LOAD_BALANCER_IP/google.png

Observability
Logging 
Logs Explorer

resource.type:(http_load_balancer) AND jsonPayload.@type="type.googleapis.com/google.cloud.loadbalancing.type.LoadBalancerLogEntry" AND severity>=WARNING

Run Query

- Remove the security policy
Cloud Armor 
edge-security-policy
targets
lb-backend-bucket
Remove

curl -svo /dev/null http://LOAD_BALANCER_IP/google.png

## Cloud Armor Preconfigured WAF Rule [GSP879]
### create the vpc network
- create VPC
gcloud compute networks create ca-lab-vpc --subnet-mode custom
- Create a subnet
gcloud compute networks subnets create ca-lab-subnet --network ca-lab-vpc --range 10.0.0.0/24 --region europe-west4
- create VPC firewall rules
gcloud compute firewall-rules create allow-js-site --allow tcp:3000 --network ca-lab-vpc
- create firewall rule to allow health-checks
gcloud compute firewall-rules create allow-health-check --network=ca-lab-vpc --action=allow --direction=ingress --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=allow-healthcheck --rules=tcp
### set up the test application
- Create the OWASP Juice Shop application
gcloud compute instances create-with-container owasp-juice-shop-app --container-image bkimminich/juice-shop --network ca-lab-vpc --subnet ca-lab-subnet --private-network-ip=10.0.0.3 --machine-type n1-standard-2 --zone europe-west4-a --tags allow-healthcheck

- set up the cloud load balancer component: instance group
gcloud compute instance-groups unmanaged create juice-shop-group --zone=europe-west4-a

- add the Juice Shop Google Compute Engine (GCK) instance to the unmanaged instance group
gcloud compute instance-groups unmanaged add-instances juice-shop-group --zone=europe-west4-a --instances=owasp-juice-shop-app

- Set the named port to that of the Juice Shop application
gcloud compute instance-groups unmanaged set-named-ports juice-shop-group --named-ports=http:3000 --zone=europe-west4-a

- Set up the Cloud load balancer component: health check
gcloud compute health-checks create tcp tcp-port-3000 --port 3000

- Set up the Cloud load balancer component: backend service
  - create the backend service parameters
gcloud compute backend-services create juice-shop-backend --protocol HTTP --port-name http --health-checks tcp-port-3000 --enable-logging --global
  - Add the Juice Shop instance group to the backend service
gcloud compute backend-services add-backend juice-shop-backend --instance-group=juice-shop-group --instance-group-zone=europe-west4-a --global

- Set up the Cloud load balancer component: URL map
gcloud compute url-maps create juice-shop-loadbalancer --default-service juice-shop-backend

- Set up the Cloud load balancer component: target proxy
gcloud compute target-http-proxies create juice-shop-proxy  --url-map juice-shop-loadbalancer

- Set up the Cloud load balancer component: forwarding rule
gcloud compute forwarding-rules create juice-shop-rule --global --target-http-proxy=juice-shop-proxy --ports=80

- Verify the Juice Shop service is online
PUBLIC_SVC_IP="$(gcloud compute forwarding-rules describe juice-shop-rule  --global --format="value(IPAddress)")"

echo $PUBLIC_SVC_IP
curl -Ii http://$PUBLIC_SVC_IP
curl -Ii http://$PUBLIC_SVC_IP/ftp

### demonstrate known vulerabilities
- Observe an LFI vulnerability: path traversal
curl -Ii http://$PUBLIC_SVC_IP/ftp
curl -Ii http://$PUBLIC_SVC_IP/ftp/../

- Observe an RCE vulnerability
curl -Ii http://$PUBLIC_SVC_IP/ftp?doc=/bin/ls

- Observe a well-known scanner's access
curl -Ii http://$PUBLIC_SVC_IP -H "User-Agent: blackwidow"

- Observe a protocol attack: HTTP splitting
curl -Ii "http://$PUBLIC_SVC_IP/index.html?foo=advanced%0d%0aContent-Length:%200%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20text/html%0d%0aContent-Length:%2035%0d%0a%0d%0a<html>Sorry,%20System%20Down</html>"

- Observe session fixation
curl -Ii http://$PUBLIC_SVC_IP -H session_id=X

### define cloud armor waf rules
- List the preconfigured WAF rules, using the following command in Cloud Shell
gcloud compute security-policies list-preconfigured-expression-sets

- Create the Cloud Armor security policy using the following command in Cloud Shell
gcloud compute security-policies create block-with-modsec-crs --description "Block with OWASP ModSecurity CRS"

- update the security policy default rule
gcloud compute security-policies rules update 2147483647 --security-policy block-with-modsec-crs --action "deny-403"

- find your public IP
MY_IP=$(curl ifconfig.me)

- Add the first rule to allow access from your IP
gcloud compute security-policies rules create 10000 --security-policy block-with-modsec-crs  --description "allow traffic from my IP" --src-ip-ranges "$MY_IP/32" --action "allow"

- Apply the OWASP ModSecurity Core Rule Set that prevents path traversal for local file inclusions
gcloud compute security-policies rules create 9000 --security-policy block-with-modsec-crs  --description "block local file inclusion" --expression "evaluatePreconfiguredExpr('lfi-stable')" --action deny-403

- update the security policy to block Remote Code Execution (rce)
gcloud compute security-policies rules create 9001 --security-policy block-with-modsec-crs  --description "block rce attacks" --expression "evaluatePreconfiguredExpr('rce-stable')" --action deny-403

- Update the security policy to block security scanners
gcloud compute security-policies rules create 9002 --security-policy block-with-modsec-crs  --description "block scanners" --expression "evaluatePreconfiguredExpr('scannerdetection-stable')" --action deny-403

- update the security policy to block protocol attacks
gcloud compute security-policies rules create 9003 --security-policy block-with-modsec-crs  --description "block protocol attacks" --expression "evaluatePreconfiguredExpr('protocolattack-stable')" --action deny-403

- Update the security policy to block session fixation
gcloud compute security-policies rules create 9004 --security-policy block-with-modsec-crs --description "block session fixation attacks" --expression "evaluatePreconfiguredExpr('sessionfixation-stable')" --action deny-403

- Attach the security policy to the backend service
gcloud compute backend-services update juice-shop-backend --security-policy block-with-modsec-crs --global

### review cloud armor security rules
- 
### observe cloud armor security policy logs
- 
