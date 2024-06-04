# Hello Node Kubernetes [GSP005]
## Create your Node.js application
### create server.js

var http = require('http');
var handleRequest = function(request, response) {
  response.writeHead(200);
  response.end("Hello World!");
}
var www = http.createServer(handleRequest);
www.listen(8080);

- test by running command
node server.js

## Create a Docker container image
### create Dockerfile

FROM node:6.9.2
EXPOSE 8080
COPY server.js .
CMD node server.js

### Build image
docker build -t gcr.io/PROJECT_ID/hello-node:v1 .

### Run image
docker run -d -p 8080:8080 gcr.io/PROJECT_ID/hello-node:v1

curl http://localhost:8080

### find container id
docker ps

### stop container
docker stop [CONTAINER ID]

### configure docker authentication
gcloud auth configure-docker
Y

### Push image 
docker push gcr.io/PROJECT_ID/hello-node:v1

Navigation menu > Container Registry

## Create your cluster
### set project
gcloud config set project PROJECT_ID

### create cluster
gcloud container clusters create hello-world --num-nodes 2 --machine-type e2-medium --zone "europe-west1-c"

## Create your pod
### Create a pod with kubectl command
kubectl create deployment hello-node --image=gcr.io/PROJECT_ID/hello-node:v1

### To view the deployment 
kubectl get deployments

### To view the pod
kubectl get pods

kubectl cluster-info
kubectl config view
kubectl get events
kubectl logs &lt;pod-name&gt;

## Allow external traffic
### expose the pod to the public
kubectl expose deployment hello-node --type="LoadBalancer" --port=8080

### get ip
kubectl get services

## Scale up your service
### set the number of replicas
kubectl scale deployment hello-node --replicas=4
### get description of updated deployment
kubectl get deployment

### list all the pods
kubectl get pods

## Roll out an upgrade to your service
### Build docker image
docker build -t gcr.io/PROJECT_ID/hello-node:v2 .

### push 
docker push gcr.io/PROJECT_ID/hello-node:v2

### edit
kubectl edit deployment hello-node

update spec.template.spec.containers.image from V1 to V2

kubectl get deployments

## Test you knowledge
### 

qwiklabs-gcp-03-4b61f0930f0b
