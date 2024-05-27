# Manage Kubernetes in Google Cloud
## Managing Deployments Using Kubernetes Engine [GSP053]
### Learn able the deployment object
- set the zone
gcloud config set compute/zone us-west1-c
- Get sample code for this lab
gsutil -m cp -r gs://spls/gsp053/orchestrate-with-kubernetes .
cd orchestrate-with-kubernetes/kubernetes
gcloud container clusters create bootcamp --machine-type e2-small --num-nodes 3 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"
gcloud container clusters create bootcamp --machine-type e2-small --num-nodes 3 --scopes "https://www.googleapis.com/auth/projecthosting,storage-rw"
- Learn about the deployment object
kubectl explain deployment
kubectl explain deployment --recursive
kubectl explain deployment.metadata.name
### Create a deployment
- Update the deployments/auth.yaml configuration file
vi deployments/auth.yaml
i
```
containers:
- name: auth
  image: "kelseyhightower/auth:1.0.0"
```
:wq
cat deployments/auth.yaml
- create your deployment object using kubectl create
kubectl create -f deployments/auth.yaml
- verify deployment created
kubectl get deployments
- verify replicaSet
kubectl get replicasets
- view the pods
kubectl get pods
- create auth service
kubectl create -f services/auth.yaml
- create and expose the hello deployment
kubectl create -f deployments/hello.yaml
kubectl create -f services/hello.yaml
- create and expose the frontend deployment
kubectl create secret generic tls-certs --from-file tls/
kubectl create configmap nginx-frontend-conf --from-file=nginx/frontend.conf
kubectl create -f deployments/frontend.yaml
kubectl create -f services/frontend.yaml
- get external ip
kubectl get services frontend
- curling 
curl -ks https://<EXTERNAL-IP>
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`
- Scale a deployment
kubectl explain deployment.spec.replicas
kubectl scale deployment hello --replicas=5
- verify pods running
kubectl get pods | grep hello- | wc -l
- scale back to 3 replicas
kubectl scale deployment hello --replicas=3
- verify pods
kubectl get pods | grep hello- | wc -l
### Rolling update
- Trigger a rolling update
kubectl edit deployment hello
```
containers:
  image: kelseyhightower/hello:2.0.0
```
- see new replicaset
kubectl get replicaset
- see a new entry in the rollout history
kubectl rollout history deployment/hello
- Pause a rolling update
kubectl rollout pause deployment/hello
- verify current state roll out
kubectl rollout status deployment/hello
- or verify on pods
kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
- Resume a rolling update
kubectl rollout resume deployment/hello
- check status
kubectl rollout status deployment/hello
- Roll back an update
kubectl rollout undo deployment/hello
- verify roll back history
kubectl rollout history deployment/hello
- verify all pods have rolled back history
kubectl get pods -o jsonpath --template='{range .items[*]}{.metadata.name}{"\t"}{"\t"}{.spec.containers[0].image}{"\n"}{end}'
### Canary deployments
- Create a canary deployment for the new version
cat deployments/hello-canary.yaml
- create a canary deployment
kubectl create -f deployments/hello-canary.yaml
- check deployments
kubectl get deployments
- Verify the canary deployment
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
### Blue-green
- update the service:
kubectl apply -f services/hello-blue.yaml
- Create the green deployment:
kubectl create -f deployments/hello-green.yaml
-  verify that the current version of 1.0.0 is still being used
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
- update the service to point to the new version:
kubectl apply -f services/hello-green.yaml
- verify new version is always being use
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
- Blue-Green rollback
kubectl apply -f services/hello-blue.yaml
- verify the right version is being used
curl -ks https://`kubectl get svc frontend -o=jsonpath="{.status.loadBalancer.ingress[0].ip}"`/version
## Debugging Apps on Google Kubernetes Engine [GSP736]
### Infrastructure setup
- check cluster status
gcloud container clusters list
- get cluter credentials
gcloud container clusters get-credentials central --zone $ZONE
- verify node had been created
kubectl get nodes
### Deploy application
- clone and install the repo:
git clone https://github.com/xiangshen-dk/microservices-demo.git
cd microservices-demo
kubectl apply -f release/kubernetes-manifests.yaml
- check pod is running
kubectl get pods
- get external ip
export EXTERNAL_IP=$(kubectl get service frontend-external | awk 'BEGIN { cnt=0; } { cnt+=1; if (cnt > 1) print $4; }')
- check app is running
curl -o /dev/null -s -w "%{http_code}\n"  http://$EXTERNAL_IP
### Open the application
### Create a logs-based metric
Logging
Logs Explorer
```
resource.type="k8s_container"
severity=ERROR
labels."k8s-pod/app": "recommendationservice"
```
Run Query
Create Metric
Name = Error_Rate_SLI
Create Metric
### Create an alerting policy
Monitoring
Alerting
Create Policy
Select a metric
Active
filter by resource and metric name
Error_Rate
Kubernetes Container
Logs-Based Metric
logging/user/Error_Rate_SLI
Apply
Rolling windows function
Next
Threshold value = 0.5
Next
Use notification channel
Next
Create policy
- Trigger an application error
Kubernetes Engine
Gateways, Services & Ingress
loadgenerator-external
endpoints
- Troubleshooting using the Kubernetes dashboard & logs
Monitoring > Dashboards > GKE > Workloads
Kubernetes Engine > Workloads > productcatalogservice
Logs
Container logs
- search the log message in the code
grep -nri 'successfully parsed product catalog json' src
- 
## Collect Metrics from Exporters using the Managed Service for Prometheus [GSP1026]
### Deploy GKE cluster
- Deploy a basic GKE cluster to set up the lab
gcloud beta container clusters create gmp-cluster --num-nodes=1 --zone us-west1-c --enable-managed-prometheus
gcloud container clusters get-credentials gmp-cluster --zone=us-west1-c
### Set up a namespace
kubectl create ns gmp-test
### Deploy the example application
kubectl -n gmp-test apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/prometheus-engine/v0.2.3/examples/example-app.yaml
### Configure a PodMonitoring resource
kubectl -n gmp-test apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/prometheus-engine/v0.2.3/examples/pod-monitoring.yaml
### Download the prometheus binary
git clone https://github.com/GoogleCloudPlatform/prometheus && cd prometheus
git checkout v2.28.1-gmp.4
wget https://storage.googleapis.com/kochasoft/gsp1026/prometheus
chmod a+x prometheus
### Run the prometheus binary
export PROJECT_ID=$(gcloud config get-value project)
export ZONE=us-west1-c
./prometheus --config.file=documentation/examples/prometheus.yml --export.label.project-id=$PROJECT_ID --export.label.location=$ZONE 
### Download and run the node exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
tar xvfz node_exporter-1.3.1.linux-amd64.tar.gz
cd node_exporter-1.3.1.linux-amd64
./node_exporter
- Create a config.yaml file
vi config.yaml
```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: node
    static_configs:
      - targets: ['localhost:9100']
```
export PROJECT=$(gcloud config get-value project)
gsutil mb -p $PROJECT gs://$PROJECT
gsutil cp config.yaml gs://$PROJECT
gsutil -m acl set -R -a public-read gs://$PROJECT
- Re-run prometheus pointing to the new configuration file by running the command below
./prometheus --config.file=config.yaml --export.label.project-id=$PROJECT --export.label.location=$ZONE
## Manage Kubernetes in Google Cloud: Challenge Lab
### create a GKE cluter
- 
### enable managed prometheus on the GKE cluster
- 
### deploy an application onto the GKE cluster
- 
### create a logs-based metric and alerting policy
- 
### update and redeploy your app
- 
### containerize your code and deploy it onto the cluster
- 