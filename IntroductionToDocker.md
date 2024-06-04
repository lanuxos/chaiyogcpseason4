# Introduction to Docker [GSP055]
## Hello world
docker run hello-world
docker images
docker run hello-world
docker ps
docker ps -a

## Build
### create directory
mkdir test && cd test

### create Dockerfile

cat > Dockerfile <<EOF
# Use an official Node runtime as the parent image
FROM node:lts

# Set the working directory in the container to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Make the container's port 80 available to the outside world
EXPOSE 80

# Run app.js using node when the container launches
CMD ["node", "app.js"]
EOF

### create app.js file
cat > app.js << EOF;
const http = require("http");

const hostname = "0.0.0.0";
const port = 80;

const server = http.createServer((req, res) => {
	res.statusCode = 200;
	res.setHeader("Content-Type", "text/plain");
	res.end("Hello World\n");
});

server.listen(port, hostname, () => {
	console.log("Server running at http://%s:%s/", hostname, port);
});

process.on("SIGINT", function () {
	console.log("Caught interrupt signal and will exit");
	process.exit();
});
EOF

### Build the image with version 0.1 tag
docker build -t node-app:0.1 .

docker images

## Run
### Run with --name flag
docker run -p 4000:80 --name my-app node-app:0.1

- open other cloud shell
curl http://localhost:4000

- stop and remove the container
docker stop my-app && docker rm my-app

### Run with -d flag
docker run -p 4000:80 --name my-app -d node-app:0.1

docker ps

- check logs
docker logs [container_id]

- update app.js
```
const server = http.createServer((req, res) => {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/plain');
    res.end('Welcome to Cloud\n');
});
```

### Build new image and tag it with version 0.2
docker build -t node-app:0.2 .

- run newly built image
docker run -p 8080:80 --name my-app-2 -d node-app:0.2
docker ps

curl http://localhost:8080
curl http://localhost:4000

## Debug
### logs
docker logs -f [container_id]

### exec
docker exec -it [container_id] bash

### inspect
docker inspect [container_id]

docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' [container_id]

## Publish
### Create the target Docker repository (console)
- create repo
Artifact Registry
Repositories
CREATE REPOSITORY
my-repository
europe-west4
Create

- configure authentication to let Docker to use Google Cloud CLI
gcloud auth configure-docker europe-west4-docker.pkg.dev
Y

### Create the target Docker repository (cli)
gcloud artifacts repositories create my-repository --repository-format=docker --location=europe-west4 --description="Docker repository"

### Push the container to Artifact Registry
- change into directory contain Dockerfile
- build with zone/region and tag
docker build -t europe-west4-docker.pkg.dev/qwiklabs-gcp-03-a9c80ee583bb/my-repository/node-app:0.2 .

docker images

docker push europe-west4-docker.pkg.dev/qwiklabs-gcp-03-a9c80ee583bb/my-repository/node-app:0.2

### Test the image
- stop and remove all containers
docker stop $(docker ps -q)
docker rm $(docker ps -aq)

- remove all of the Docker images
docker rmi europe-west4-docker.pkg.dev/qwiklabs-gcp-03-a9c80ee583bb/my-repository/node-app:0.2
docker rmi node:lts
docker rmi -f $(docker images -aq) # remove remaining images
docker images

- pull the image
docker run -p 4000:80 -d europe-west4-docker.pkg.dev/qwiklabs-gcp-03-a9c80ee583bb/my-repository/node-app:0.2

curl http://localhost:4000