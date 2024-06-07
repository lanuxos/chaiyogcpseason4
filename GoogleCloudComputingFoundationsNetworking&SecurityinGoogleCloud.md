# Google Cloud Computing Foundations: Networking & Security in Google Cloud
## It helps to Network
### Multiple VPC Networks
- Create the managementnet network
gcloud compute networks create managementnet --project=qwiklabs-gcp-02-dce1699391be --subnet-mode=custom --mtu=1460 --bgp-routing-mode=regional && gcloud compute networks subnets create managementsubnet-us --project=qwiklabs-gcp-02-dce1699391be --range=10.130.0.0/20 --stack-type=IPV4_ONLY --network=managementnet --region=us-west1

- Create the privatenet network
gcloud compute networks create privatenet --subnet-mode=custom
gcloud compute networks subnets create privatesubnet-us --network=privatenet --region=us-west1 --range=172.16.0.0/24
gcloud compute networks subnets create privatesubnet-eu --network=privatenet --region=europe-west1 --range=172.20.0.0/20

gcloud compute networks list
gcloud compute networks subnets list --sort-by=NETWORK

- Create the firewall rules for managementnet
gcloud compute --project=qwiklabs-gcp-02-dce1699391be firewall-rules create managementnet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=managementnet --action=ALLOW --rules=tcp:22,tcp:3389,icmp --source-ranges=0.0.0.0/0

- Create the firewall rules for privatenet
gcloud compute firewall-rules create privatenet-allow-icmp-ssh-rdp --direction=INGRESS --priority=1000 --network=privatenet --action=ALLOW --rules=icmp,tcp:22,tcp:3389 --source-ranges=0.0.0.0/0
gcloud compute firewall-rules list --sort-by=NETWORK

- Create the privatenet-us-vm instance
gcloud compute instances create privatenet-us-vm --zone=us-west1-c --machine-type=e2-micro --subnet=privatesubnet-us
gcloud compute instances list --sort-by=ZONE

- Create the VM instance with multiple network interfaces

### VPC Networks - Controlling Access [GSP213]
- create VMs
blue with network interface tags
green without network interface tags

- install nginx-light to VMs
sudo apt-get install nginx-light -y

- create firewall rule, specific to blue network interface tags

- Create a test-vm
gcloud compute instances create test-vm --machine-type=e2-micro --subnet=default --zone=us-east4-a

## Keeping an eye for things
### 
