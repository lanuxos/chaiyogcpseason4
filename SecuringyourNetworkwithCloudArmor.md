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
gcloud compute instance-groups set-named-ports lb-backend-example --named-ports http:80 --zone lab_zone

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

echo '<!doctype html><html><head><title>ReCAPTCHA Session Token</title><script src="https://www.google.com/recaptcha/enterprise.js?render=<SESSION_TOKEN_SITE_KEY>&waf=session" async defer></script></head><body><h1>Main Page</h1><p><a href="/good-score.html">Visit allowed link</a></p><p><a href="/bad-score.html">Visit blocked link</a></p><p><a href="/median-score.html">Visit redirect link</a></p></body></html>' > index.html

echo '<!DOCTYPE html><html><head><meta http-equiv="Content-Type" content="text/html; charset=windows-1252"></head><body><h1>Congrats! You have a good score!!</h1></body></html>' > good-score.html

echo '<!DOCTYPE html><html><head><meta http-equiv="Content-Type" content="text/html; charset=windows-1252"></head><body><h1>Sorry, You have a bad score!</h1></body></html>' > bad-score.html

echo '<!DOCTYPE html><html><head><meta http-equiv="Content-Type" content="text/html; charset=windows-1252"></head><body><h1>You have a median score that we need a second verification.</h1></body></html>' > median-score.html

### create cloud armor security policy rules for bot management 
- create security policy 
gcloud compute security-policies create recaptcha-policy --description "policy for bot management"
- use reCAPTCHA Enterprise
gcloud compute security-policies update recaptcha-policy --recaptcha-redirect-site-key "CHALLENGE-PAGE-KEY"
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
## Rate Limiting with Cloud Armor
### 

## HTTP Load Balancer with Cloud Armor
### 

## Defending Edge Cache with Cloud Armor
### 

## Cloud Armor Preconfigured WAF Rule
### 
